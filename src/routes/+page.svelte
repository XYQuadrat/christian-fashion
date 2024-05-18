<script lang="ts">
	import { onMount } from 'svelte';

	type StrapiImage = {
		data: string;
	};

	type Url = string;

	type BrandAttributes = {
		Name: string;
		Description: string;
		Link: Url;
		Logo: StrapiImage;
		Covers: StrapiImage;
		Status: string;
	};

	type Brand = {
		attributes: BrandAttributes;
	};

	let brands: Brand[] = [];
	let error: Error | null = null;

	onMount(async () => {
		const parseJSON = (resp: Response): Promise<Response> => resp.json();
		const checkStatus = (resp: Response) => {
			if (resp.status >= 200 && resp.status < 300) {
				return resp;
			}
			return parseJSON(resp).then((resp: Response) => {
				throw resp;
			});
		};

		const headers = {
			'Content-Type': 'application/json'
		};

		try {
			const res = await fetch('https://fashion.xyquadrat.ch/data/api/brands?populate=*', {
				method: 'GET',
				headers: {
					'Content-Type': 'application/json'
				}
			})
				.then(checkStatus)
				.then(parseJSON);

			brands = res.data;
		} catch (e) {
			error = e as Error;
		}

		brands.sort(sortItems);
	});

	function sortItems(a: Brand, b: Brand): number {
		const statusA = a.attributes.Status;
		const statusB = b.attributes.Status;
		const priorityA = statusA == 'Aktiv' ? 2 : statusA == 'Unklar' ? 1 : 0;
		const priorityB = statusB == 'Aktiv' ? 2 : statusB == 'Unklar' ? 1 : 0;

		return Number(priorityA < priorityB);
	}
</script>

{#if error !== null}
	{error}
{:else}
	<div
		class="full min-w-full flex flex-row justify-between items-center sticky bg-zinc-50/95 top-0 border-b border-zinc-800 z-10"
	>
		<h1 class="text-left font-display text-3xl tracking-tight m-4">
			<span class="font-extralight">gospel</span>
			<span
				class="font-extrabold underline-offset-2 underline"
				style="text-decoration-thickness: 4px;">garments</span
			>
		</h1>
		<a href="#">
			<button class="border-zinc-500 border p-2 font-display font-light mr-4 hover:bg-zinc-300">
				Brand vorschlagen
			</button>
		</a>
	</div>
	<div class="container font-display" style="">
		<div class="grid full min-w-full grid-cols-1 xl:grid-cols-2 gap-2 my-4">
			{#each brands as brand, i}
				<div class="flex md:flex-row flex-col min-h-96 m-2 border border-zinc-600 bg-slate-50">
					<div class="flex-none md:w-64 md:h-full h-64 relative p-3 pr-0">
						{#if brand.attributes.Covers.data != null}
							<img
								src="https://fashion.xyquadrat.ch/data{brand.attributes.Covers.data[0].attributes
									.url}"
								class="inset-0 w-full h-full object-cover"
								alt="Cover image for {brand.attributes.Name}"
							/>
						{/if}
					</div>

					<div class="flex flex-col flex-auto md:px-6 px-4 py-6">
						{#if brand.attributes.Logo.data != null}
							<img
								src="https://fashion.xyquadrat.ch/data{brand.attributes.Logo.data.attributes.url}"
								class="md:h-8 h-16 mb-4 self-center brightness-0"
								alt="Logo of {brand.attributes.Name}"
							/>
						{/if}
						<div class="flex items-center">
							<h1
								class="flex-auto md:text-4xl text-3xl font-serif font-extralight tracking-tight text-black text-center"
							>
								{brand.attributes.Name}
							</h1>
						</div>
						<div class="grow">
							<p class="my-4 max-w-3xl text-left">
								{brand.attributes.Description ?? ''}
							</p>
						</div>
						<div class="flex justify-between items-center content-center">
							{#if brand.attributes.Link != null}
								<a
									class="relative z-10 bg-indigo-500 hover:bg-indigo-600 font-semibold text-white px-4 py-3 inline-flex items-center content-center"
									href={brand.attributes.Link}
									target="_blank"
								>
									<svg
										xmlns="http://www.w3.org/2000/svg"
										fill="none"
										viewBox="0 0 24 24"
										stroke-width="2"
										stroke="currentColor"
										aria-hidden="true"
										class="w-5 h-5 me-2 inline"
									>
										<path
											stroke-linecap="round"
											stroke-linejoin="round"
											d="M13.19 8.688a4.5 4.5 0 0 1 1.242 7.244l-4.5 4.5a4.5 4.5 0 0 1-6.364-6.364l1.757-1.757m13.35-.622 1.757-1.757a4.5 4.5 0 0 0-6.364-6.364l-4.5 4.5a4.5 4.5 0 0 0 1.242 7.244"
										/>
									</svg>
									<span>Zum Shop</span>
								</a>
							{/if}
							{#if brand.attributes.Status != null}
								<div>
									<span>Status:</span>
									{#if brand.attributes.Status == 'Aktiv'}
										<span class="font-bold bg-green-400 py-1 px-2 w-9 text-white"
											>{brand.attributes.Status}</span
										>
									{/if}
									{#if brand.attributes.Status == 'Inaktiv'}
										<span class="font-bold bg-red-400 py-1 px-2 text-white"
											>{brand.attributes.Status}</span
										>
									{/if}
									{#if brand.attributes.Status == 'Unklar'}
										<span class="font-bold bg-orange-400 py-1 px-2 text-white"
											>{brand.attributes.Status}</span
										>
									{/if}
								</div>
							{/if}
						</div>
					</div>
				</div>
			{/each}
		</div>
	</div>
{/if}

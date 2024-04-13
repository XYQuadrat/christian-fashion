<script lang="ts">
	import { onMount } from 'svelte';

	type BrandAttributes = {
		Name: string;
		Description: string;
	};

	type Brand = {
		attributes: BrandAttributes;
	};

	let brands: Brand[] = [];
	let error: Error | null = null;

	onMount(async () => {
		const parseJSON = (resp: Response): Promise<Response> => (resp.json ? resp.json() : resp);
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
			const res = await fetch('http://localhost:1337/api/brands?populate=*', {
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
	});
</script>

{#if error !== null}
	{error}
{:else}
	<div class="full min-w-full flex flex-row justify-center">
		<div class="flex flex-col gap-6 py-4">
			{#each brands as brand}
				<div
					class="max-w-sm p-6 bg-white border border-gray-200 rounded-lg shadow hover:bg-gray-100"
				>
					<h1 class="text-lg font-bold">
						{brand.attributes.Name}
					</h1>
					<p>
						{brand.attributes.Description ?? ''}
					</p>
					{#if brand.attributes.Logo.data != null}
						<img src="http://localhost:1337{brand.attributes.Logo.data.attributes.url}" />
					{/if}
				</div>
			{/each}
		</div>
	</div>
{/if}

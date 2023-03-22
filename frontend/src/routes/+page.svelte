<script>
	import { fade, fly } from 'svelte/transition';

	let files = [];
	let consent = true;
	let name = '';

	let toast = {
		show: false,
		message: 'Operation successful!',
		type: 'success'
	};

	let response = {};

	function sendFiles() {
		const formdata = new FormData();
		for (let file of files) {
			formdata.append('files', file);
		}
		formdata.append('name', name);
		formdata.append('consent', consent);

		fetch('http://127.0.0.1:8000/predict', {
			method: 'POST',
			body: formdata
		})
			.then((res) => {
				if (res.ok) {
					return res.json();
				} else {
					throw new Error('Something went wrong');
				}
			})
			.then((res) => {
				console.log(res);
				response = res;
				handleToasts(true);
			})
			.catch((err) => {
				response.error = err.message;
			});
	}

	function handleToasts(
		show,
		{ message, type, delay } = {
			message: 'Operation successful!',
			type: 'success',
			delay: 3000
		}
	) {
		if (show) {
			toast.show = true;
			toast.message = message;
			toast.type = type;
			setTimeout(() => {
				toast.show = false;
			}, delay);
		}
	}

	$: if (files.length > 20) {
		files = files.slice(0, 20);
		handleToasts(true, {
			message: 'Maximum 20 images allowed',
			type: 'error',
			delay: 2000
		});
	}

</script>

<navbar class="navbar bg-base-100">
	<div class="flex-1">
		<div class="btn normal-case text-xl">Image Classifier</div>
	</div>
	<div class="flex-none">
		<button class="btn btn-square btn-ghost">
			<svg
				xmlns="http://www.w3.org/2000/svg"
				fill="none"
				viewBox="0 0 24 24"
				class="inline-block w-5 h-5 stroke-current"
				><path
					stroke-linecap="round"
					stroke-linejoin="round"
					stroke-width="2"
					d="M5 12h.01M12 12h.01M19 12h.01M6 12a1 1 0 11-2 0 1 1 0 012 0zm7 0a1 1 0 11-2 0 1 1 0 012 0zm7 0a1 1 0 11-2 0 1 1 0 012 0z"
				/></svg
			>
		</button>
	</div>
</navbar>

<div
	class="flex flex-col items-center justify-center max-w-5xl mx-auto bg-base-200 p-5 rounded-box mb-5"
>
	{#if files && files.length > 0}
		<div class="carousel carousel-center p-4 space-x-4 bg-neutral rounded-box">
			{#each files as file, i}
				<div id={`item-${i + 1}`} class="carousel-item w-full">
					<img src={URL.createObjectURL(file)} class="rounded-box w-full h-64" />
				</div>
			{/each}
		</div>
		<div class="flex justify-center w-full py-2 gap-2">
			{#each files as file, i}
				<a href={`#item-${i + 1}`} class="btn btn-xs">{i + 1}</a>
			{/each}
		</div>
	{:else}
		No images selected
	{/if}
</div>

<div class="flex items-center justify-center max-w-5xl mx-auto">
	<div class="mockup-window border bg-base-300 w-full max-h-4xl">
		<div class="flex flex-col items-center justify-center gap-3 px-4 py-16 bg-base-200">
			<div class="flex flex-col items-center justify-center gap-2">
				<input type="file" bind:files multiple class="file-input" />
				{#if files.length}
					<input
						type="text"
						bind:value={name}
						placeholder="Your name here (optional)"
						class="input w-full max-w-xs"
					/>
				{/if}

				<div
					class={`alert ${
						consent ? 'alert-success' : 'alert-error'
					} shadow-lg text-black rounded-lg`}
				>
					<label class="label cursor-pointer gap-2">
						<input type="checkbox" class="toggle toggle-success" bind:checked={consent} />
						{consent ? '' : 'Do not'} share my images!
					</label>
				</div>
				<button
					class={`btn btn-primary w-full max-w-xs ${files.length ? '' : 'btn-disabled'}`}
					on:click={sendFiles}
				>
					Submit {name ? `as ${name}` : ''}</button
				>
			</div>
		</div>

		<div class="divider">
			{#if response.results}
				<div class="btn btn-info btn-xs">
					{response.results.bike.count ?? 0} bikes and {response.results.car.count ?? 0} cars detected
				</div>
			{:else}
				<div class={`btn btn-xs ${files.length ? 'btn-success' : ''}`}>
					{files.length ? files.length : 'No'} file(s) selected
				</div>
			{/if}
		</div>

		{#if files.length}
			<div class="flex flex-col items-center justify-between px-4 py-16 bg-base-200">
				{#if response.results}
					<div class="grid grid-cols-3 gap-4">
						{#each files as file}
							<div
								class={`grid h-auto card ${
									response?.results?.bike.files.includes(file.name) ? 'bg-green-300' : 'bg-blue-300'
								} rounded-box place-items-center p-2`}
							>
								<div class="flex flex-col items-center justify-center">
									<div class="badge font-bold mb-2 rounded-lg">
										{response?.results?.bike.files.includes(file.name) ? 'bike' : 'car'}
									</div>
									<img
										class="avatar rounded-xl h-64 w-64"
										src={URL.createObjectURL(file)}
										alt={file.name}
									/>
								</div>
							</div>
						{/each}
					</div>
				{:else if response.error}
					<div class="alert alert-error">
						{response.error}
					</div>
				{:else}
					<div class="grid grid-cols-3 gap-4">
						{#each files as file}
							<div class={`grid h-auto card bg-base-100 rounded-box place-items-center p-2`}>
								<img
									class="avatar rounded-xl h-64 w-64"
									src={URL.createObjectURL(file)}
									alt={file.name}
								/>
								<button
									class="btn btn-xs btn-error btn-circle absolute top-1 left-1"
									on:click={() => {
										files = Object.assign([], files);
										files.splice(i, 1);
									}}
								>
									<svg
										xmlns="http://www.w3.org/2000/svg"
										fill="none"
										viewBox="0 0 24 24"
										class="inline-block w-5 h-5 stroke-current"
										><path
											stroke-linecap="round"
											stroke-linejoin="round"
											stroke-width="2"
											d="M6 18L18 6M6 6l12 12"
										/></svg
									>
								</button>
							</div>
						{/each}
					</div>
				{/if}
			</div>
		{/if}
	</div>
</div>

{#if toast.show}
	<div class="toast" in:fly={{ y: 200, duration: 2000 }} out:fade>
		<div class={`alert ${toast.type == "success" ? "alert-success" : "alert-error"}`}>
			<div>
				<span>{toast.message}</span>
			</div>
		</div>
	</div>
{/if}

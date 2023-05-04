<template>
	<main class='h-screen'>
		<h1 class='pt-11 absolute flex items-center justify-center gap-x-2 w-full text-center'>
			<img src="https://nacholabraweb.000webhostapp.com/ico.png">
			CreateIcons
		</h1>

		<!-- Paste Zone -->
		<label for='focusedInput' class="flex flex-col justify-center h-screen cursor-pointer" v-if="!image1">
			<div class="items-center flex flex-col">
				<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-8 h-8 text-gray-500">
					<path
						stroke-linecap="round"
						stroke-linejoin="round"
						d="M12 16.5V9.75m0 0l3 3m-3-3l-3 3M6.75 19.5a4.5 4.5 0 01-1.41-8.775 5.25 5.25 0 0110.233-2.33 3 3 0 013.758 3.848A3.752 3.752 0 0118 19.5H6.75z"
					/>
				</svg>

				<h2 class="mt-1 font-medium tracking-wide text-gray-700">Payment File</h2>
				<p class="mt-2 text-xs tracking-wide text-gray-500">Upload or paste your image</p>

				<div class="text-center">
					<p class="mt-8 font-medium tracking-wide text-gray-700">Size</p>
					<input v-model='size' class="mt-2 text-center font-medium text-gray-700" placeholder="Size Icon"/>
				</div>
			</div>

			<input type="file" accept="image/*" id='focusedInput' class="hidden" @change="changeImg" />
		</label>

		<div v-else class='grid place-content-center items-center mx-auto h-screen grid-cols-2 w-10/12'>
			<div>
				<img class="mx-auto rounded-xl" :src="image1" />
				<p class='text-center mt-8 text-gray-800'>Imagen original</p>
			</div>

			<div class="text-center cursor-pointer">
				<img class="mx-auto rounded-xl" onclick='document.getElementById("a").click()' :src="image2" />
				<a :href='image2' download id='a' class='mt-2 block text-sky-500'>Descargar</a>
				<p @click='image1 = false' class='mt-2'>Empezar de nuevo</p>
			</div>
		</div>
	</main>
</template>

<script setup>
	import { ref } from 'vue'
	import ImageTools from '@/redim.js'

	const image2 = ref()
	const image1 = ref()
	const size = ref(32)

	document.onpaste = (pasteEvent) => {
		const event = pasteEvent.clipboardData.items[0]

		if (event.type.indexOf('image') === 0) {
			const blob = event.getAsFile()
			const reader = new FileReader()

			reader.onload = (event) => image1.value = event.target.result
			reader.readAsDataURL(blob)

	    resizeTools(blob, size.value)
		}
	}

	const changeImg = (event) => {
	  const reader = new FileReader()

	  reader.onload = (event) => image1.value = event.target.result
	  reader.readAsDataURL(event.target.files[0])

    resizeTools(event.target.files[0], size.value)
	}

	const resizeTools = (event, calidad) => {
		ImageTools.resize(event, { width: calidad, height: calidad }, (blob, didItResize) => {
      const reader = new FileReader()

      reader.onload = (event) => image2.value = event.target.result
      reader.readAsDataURL(blob)
		})
	}
</script>
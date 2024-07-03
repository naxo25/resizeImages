<template>
	<main class='h-screen'>
		<h1 class='pt-11 absolute flex items-center justify-center gap-x-2 w-full text-center'>
			<img src="/ico.png">
			resizeImages
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
					<p class="mt-8 font-medium tracking-wide text-lg text-gray-700">New Size of image/icon</p>
					<input v-model='size' class="mt-2 border py-0.5 px-3 rounded text-center font-medium text-gray-700" :placeholder="'default' + sizeDefault"/>
				</div>
			</div>

			<input type="file" accept="image/*" id='focusedInput' class="hidden" @change="changeImg" />
		</label>

		<div v-else class='grid place-content-center items-center mx-auto h-screen grid-cols-2 w-10/12'>
			<div>
				<img id='img1' class="mx-auto rounded" :src="image1" />
				<p class='text-center mt-8 text-gray-800'>Imagen original</p>
			</div>

			<div class="text-center cursor-pointer">
				<div v-if='loader'>
					loading ...
				</div>
				<img v-else class="mx-auto rounded" onclick='document.getElementById("a").click()' :src="image2" />
				<div class="flex justify-center">
					<a :href='image2' download id='a' class='mt-2 text-sm font-medium block text-sky-500'>Descargar imagen</a>
					<a :href='image2' target="_blank" class='mt-2 ml-1 block text-sm font-medium text-green-900'>Abrir en otra pestaña</a>
				</div>
				<p @click='copyData' class='mt-2 block text-sm font-medium text-sky-500'>Copiar blob url</p>
				<p @click='image1 = false' class='mt-2 text-sm font-medium'>Empezar de nuevo</p>
			</div>
		</div>
	</main>
	<Toaster position="bottom-right" />
</template>

<script setup>
	import { ref } from 'vue'
	import { Toaster, toast } from 'vue-sonner'
	import ImageTools from '@/redim.js'

	import imageCompression from 'browser-image-compression';

	async function convertToAVIF(file) {
		const options = {
			maxSizeMB: 1,
			maxWidthOrHeight: 1920,
			useWebWorker: true
		}
		try {
			const compressedFile = await imageCompression(file, options);
			return compressedFile;
		} catch (error) {
			console.log(error);
		}
	}

	const image2 = ref()
	const image1 = ref()
	const sizeDefault = 300
	const size = ref()
	const loader = ref(false)

	document.onpaste = async pasteEvent => {
		const event = pasteEvent.clipboardData.items[0]

		if (event.type.indexOf('text') === 0) {
			image1.value = pasteEvent.clipboardData.getData('text')
			return
		}

		if (event.type.indexOf('image') === 0) {
			const blob = event.getAsFile()
			const reader = new FileReader()

			reader.onload = (event) => image1.value = event.target.result
			reader.readAsDataURL(blob)

			loader.value = true
	    resizeTools(blob, size.value || sizeDefault)
			loader.value = false
		}
	}

	const changeImg = async event => {
	  const reader = new FileReader()
	  const binaryImg = event.target.files[0]

	  reader.onload = (event) => image1.value = event.target.result
	  reader.readAsDataURL(binaryImg)

    resizeTools(binaryImg, size.value || sizeDefault)
	}

	const resizeTools = (event, calidad) => {
		ImageTools.resize(event, { width: calidad, height: calidad }, async (blob, didItResize) => {
      const avifBinary = await convertToAVIF(blob);
      image2.value = URL.createObjectURL(avifBinary)
		})
	}

	const copyData = () => {
		navigator.clipboard.writeText(image2.value);
		toast("Contenido copiado al portapapeles.");
	}
</script>

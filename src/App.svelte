<script>

	import { onMount } from 'svelte';
	import handleError from './Helpers/handleError.js';
	import Error from './UI/Error.svelte';

	// pages
	// ------------------------------------------------------------------------
	import Hello from './Screens/Hello.svelte';
	import Test from './Screens/Test.svelte';
	import Home from './Screens/Home.svelte';
	import Training from './Screens/Training.svelte';
	import Finish from './Screens/Finish.svelte';
	import News from './Screens/News.svelte';

	// submenus
	// ------------------------------------------------------------------------
	import Main from './Menu/Main.svelte';
	import ResetApp from './Menu/ResetApp.svelte';
	import ManualTrainingLevel from './Menu/ManualTrainingLevel.svelte';
	import Instructions from './Menu/Instructions.svelte'
	import PrivacyPolicy from './Menu/PrivacyPolicy.svelte';
	import CancelTraining from './Menu/CancelTraining.svelte';
	import ManualLanguage from './Menu/ManualLanguage.svelte';

	// stores
	// ------------------------------------------------------------------------
	import state from './Data/state.js';
	import translations from './Data/translations.js';


	// app possible states
	// ------------------------------------------------------------------------
	let pages = {Hello, Test, Home, Training, Finish};
	let submenus = {Main, ResetApp, ManualTrainingLevel, Instructions, PrivacyPolicy, CancelTraining, ManualLanguage};


	// if iOS app in fullscreen mode add class
	// ------------------------------------------------------------------------
	if(window.navigator.standalone === true){
		document.documentElement.classList.add('app-appleFullscreen');
	}


	// enable 'add to home screen' on chrome mobile
	// ------------------------------------------------------------------------
	// register the (empty) service worker
	if('serviceWorker' in navigator){
		navigator.serviceWorker.register('/service-worker.js')
			.catch((error) => {
				console.log('Service worker registration failed.', error);
			});
	};

	// cache the install prompt
	let deferredInstallPrompt;

	window.addEventListener('beforeinstallprompt', (event) => {
		// prevent the infobar from appearing on mobile
		event.preventDefault();
		// save the event to trigger it later
		deferredInstallPrompt = event;
	});

	// analytics
	window.addEventListener('appinstalled', () => {
		gtag('event', 'app', {'mobileInstalled': 1});
	});

	if($state.page === 'Home' && localStorage.installPromptShown === 'false' && deferredInstallPrompt){
		deferredInstallPrompt.prompt();
		localStorage.installPromptShown = true;

		// analytics
		gtag('event', 'app', {'mobileInstallInfoShown': 1});
	}


	// if user came here after a day, reset his training
	// ------------------------------------------------------------------------
	if(Date.now() - $state.lastActive > 86400000){
		state.setStep(0)
		state.isResting(0);
		state.setPage('Home');
	}

</script>




<News />

<main
	class="app"
	class:app-unfocused="{ $state.popupShown || $state.submenu }"
	class:app-menuShown="{ $state.submenu }"
>
	<svelte:component this="{ pages[$state.page] }" />
</main>

{#if $state.submenu}
	<svelte:component this="{ submenus[$state.submenu] }" />
{/if}

<Error />
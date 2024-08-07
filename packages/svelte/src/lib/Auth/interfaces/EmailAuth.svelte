<script>
	import Anchor from '../../UI/Anchor.svelte';
	import Button from '../../UI/Button.svelte';
	import Container from '../../UI/Container.svelte';
	import Input from '../../UI/Input.svelte';
	import Label from '../../UI/Label.svelte';
	import Message from '../../UI/Message.svelte';
	import { VIEWS } from '@supabase/auth-ui-shared';
	export let authView = 'sign_in';
	export let email = '';
	export let password = '';
	export let supabaseClient;
	export let redirectTo = undefined;
	export let additionalData = undefined;
	export let showLinks = false;
	export let magicLink = true;
	export let i18n;
	export let appearance;
	let message = '';
	let error = '';
	let loading = false;
	let lngKey = authView === 'sign_in' ? 'sign_in' : 'sign_up';

	async function handleSubmit() {
		loading = true;
		error = '';
		message = '';
		switch (authView) {
			case VIEWS.SIGN_IN:
				const { error: signInError } = await supabaseClient.auth.signInWithPassword({
					email,
					password
				});
				if (signInError)
					error = signInError.message;
				loading = false;
				break;
			case VIEWS.SIGN_UP:
				if (grecaptcha.getResponse() === '') {
					error = 'Please complete the reCAPTCHA';
					break;
				}
				let options = {
					emailRedirectTo: redirectTo
				};
				if (additionalData) {
					options.data = additionalData;
				}
				const { data: fetchData, error: fetchError } = await supabaseClient
					.from('users')
					.select('email')
					.eq('email', email)
				if (fetchData.length > 0) {
					error = 'An account with this email already exists, please sign in or reset your password';
				} else {
					const { data: { user: signUpUser, session: signUpSession }, error: signUpError } = await supabaseClient.auth.signUp({
						email,
						password,
						options
					});
					if (signUpError)
						error = signUpError.message;
					// Check if session is null -> email confirmation setting is turned on
					else if (signUpUser && !signUpSession)
						message = i18n.sign_up?.confirmation_text;
				}
		}
		grecaptcha.reset();
		loading = false;
	}
</script>

<svelte:head>
	<script src="https://www.google.com/recaptcha/api.js" async defer></script>
</svelte:head>

<form method="post" on:submit|preventDefault={handleSubmit}>
	<Container direction="vertical" gap="large" {appearance}>
		<Container direction="vertical" gap="large" {appearance}>
			<div>
				<Label for="email" {appearance}>{i18n?.[lngKey]?.email_label}</Label>
				<Input
					id="email"
					type="email"
					name="email"
					autofocus
					placeholder={i18n?.[lngKey]?.email_input_placeholder}
					bind:value={email}
					autocomplete="email"
					{appearance}
				/>
			</div>
			<div>
				<Label for="password" {appearance}>{i18n?.[lngKey]?.password_label}</Label>
				<Input
					id="password"
					type="password"
					name="password"
					placeholder={i18n?.[lngKey]?.password_input_placeholder}
					bind:value={password}
					autocomplete={authView === VIEWS.SIGN_IN ? 'current-password' : 'new-password'}
					{appearance}
				/>
			</div>
			<slot />
		</Container>
		{#if authView === VIEWS.SIGN_UP}
			<div class="g-recaptcha" data-sitekey="6Le2OSAqAAAAAPhgFtFJ0jZYE54phYmfV6262KJ3"></div>
		{/if}
		<Button type="submit" color="primary" {loading} {appearance}
			>{loading ? i18n?.[lngKey]?.loading_button_label : i18n?.[lngKey]?.button_label}
		</Button>

		{#if showLinks}
			<Container direction="vertical" gap="small" {appearance}>
				{#if authView === VIEWS.SIGN_IN && magicLink}
					<Anchor
						on:click={(e) => {
							e.preventDefault();
							authView = VIEWS.MAGIC_LINK;
						}}
						href="#auth-magic-link"
						{appearance}
						>{i18n?.magic_link?.link_text}
					</Anchor>
				{/if}
				{#if authView === VIEWS.SIGN_IN}
					<Anchor
						on:click={(e) => {
							e.preventDefault();
							authView = VIEWS.FORGOTTEN_PASSWORD;
						}}
						href="#auth-forgot-password"
						{appearance}
					>
						{i18n?.forgotten_password?.link_text}</Anchor
					>
					<Anchor
						on:click={(e) => {
							e.preventDefault();
							authView = VIEWS.SIGN_UP;
						}}
						href="#auth-sign-up"
						{appearance}
					>
						{i18n?.sign_up?.link_text}
					</Anchor>
				{:else}
					<Anchor
						on:click={(e) => {
							e.preventDefault();
							authView = VIEWS.SIGN_IN;
						}}
						href="#auth-sign-in"
						{appearance}
					>
						{i18n?.sign_in?.link_text}
					</Anchor>
				{/if}
			</Container>
		{/if}
	</Container>

	{#if message}
		<Message {appearance}>
			{message}
		</Message>
	{/if}
	{#if error}
		<Message color="danger" {appearance}>
			{error}
		</Message>
	{/if}
</form>

<style>
	form {
		width: 100%;
	}
	.g-recaptcha {
		margin: 0 auto;
	}
</style>

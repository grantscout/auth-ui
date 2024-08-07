<script lang="ts">
	import type { SupabaseClient } from '@supabase/supabase-js';
	import Anchor from '../../UI/Anchor.svelte';
	import Button from '../../UI/Button.svelte';
	import Container from '../../UI/Container.svelte';
	import Input from '../../UI/Input.svelte';
	import Label from '../../UI/Label.svelte';
	import Message from '../../UI/Message.svelte';
	import { VIEWS, type I18nVariables, type ViewType } from '@supabase/auth-ui-shared';
	import type { Appearance } from '../../types';

	export let i18n: I18nVariables;
	export let supabaseClient: SupabaseClient;
	export let authView: ViewType;
	export let redirectTo: string | undefined = undefined;
	export let email = '';
	export let showLinks = false;
	export let appearance: Appearance;

	let message = '';
	let error = '';
	let loading = false;

	async function handleSubmit() {
		loading = true;
		error = '';
		message = '';
		if (grecaptcha.getResponse() === '') {
			error = 'Please complete the reCAPTCHA';
		} else {
			const { error: resetPasswordError } = await supabaseClient.auth.resetPasswordForEmail(email, {
				redirectTo
			});
			if (resetPasswordError) error = resetPasswordError.message;
			else message = i18n.forgotten_password?.confirmation_text as string;
			grecaptcha.reset();
		}
		loading = false;
	}
</script>

<svelte:head>
	<script src="https://www.google.com/recaptcha/api.js" async defer></script>
</svelte:head>

<form id="auth-forgot-password" method="post" on:submit|preventDefault={handleSubmit}>
	<Container direction="vertical" gap="large" {appearance}>
		<Container direction="vertical" gap="large" {appearance}>
			<div>
				<Label for="email" {appearance}>{i18n?.forgotten_password?.email_label}</Label>
				<Input
					id="email"
					type="email"
					name="email"
					autofocus
					placeholder={i18n?.forgotten_password?.email_input_placeholder}
					bind:value={email}
					autocomplete="email"
					{appearance}
				/>
			</div>
			<div class="g-recaptcha" data-sitekey="6Le2OSAqAAAAAPhgFtFJ0jZYE54phYmfV6262KJ3"></div>
			<Button type="submit" color="primary" {loading} {appearance}>
				{loading ? i18n?.forgotten_password?.loading_button_label : i18n?.forgotten_password?.button_label}
			</Button>
		</Container>

		{#if showLinks}
			<Anchor
				on:click={(e) => {
					e.preventDefault();
					authView = VIEWS.SIGN_IN;
				}}
				href="#auth-magic-link"
				{appearance}>{i18n?.sign_in?.link_text}</Anchor
			>
		{/if}
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
	</Container>
</form>

<style>
	form {
		width: 100%;
	}
	.g-recaptcha {
		margin: 0 auto;
	}
</style>

<script>import { createStitches, createTheme } from '@stitches/core';
import { merge, VIEWS, en } from '@supabase/auth-ui-shared';
import EmailAuth from './interfaces/EmailAuth.svelte';
import ForgottenPassword from './interfaces/ForgottenPassword.svelte';
import MagicLink from './interfaces/MagicLink.svelte';
import SocialAuth from './interfaces/SocialAuth.svelte';
import UpdatePassword from './interfaces/UpdatePassword.svelte';
import VerifyOtp from './interfaces/VerifyOtp.svelte';
import { onMount } from 'svelte';
export let supabaseClient;
export let socialLayout = 'vertical';
export let providers = [];
export let providerScopes = undefined;
export let queryParams = undefined;
export let view = 'sign_in';
export let redirectTo = undefined;
export let onlyThirdPartyProviders = false;
export let magicLink = false;
export let showLinks = true;
export let appearance = {};
export let theme = 'default';
export let localization = {};
export let otpType = 'email';
export let additionalData = undefined;
onMount(() => {
    const { data: authListener } = supabaseClient.auth.onAuthStateChange((event) => {
        if (event === 'SIGNED_IN') {
            if (view === VIEWS.FORGOTTEN_PASSWORD) {
				view = 'update_password';
			}
        }
        else if (event === 'USER_UPDATED') {
            view = 'sign_in';
        }
    });
    () => authListener.subscription.unsubscribe();
});
$: i18n = merge(en, localization.variables ?? {});
$: createStitches({
    theme: merge(appearance?.theme?.default ?? {}, appearance?.variables?.default ?? {})
});
$: themeVariables = createTheme(merge(
// @ts-ignore
appearance?.theme?.[theme], appearance?.variables?.[theme] ?? {}));
/**
 * Simple boolean to detect if view 'sign_in' or 'sign_up' or 'magic_link' is used
 *
 * @returns boolean
 */
$: SignView = view === 'sign_in' || view === 'sign_up' || view === 'magic_link';
</script>

<div class={theme !== 'default' ? themeVariables : ''}>
	{#if SignView}
		<SocialAuth
			{appearance}
			{supabaseClient}
			{providers}
			{providerScopes}
			{queryParams}
			{socialLayout}
			{redirectTo}
			{onlyThirdPartyProviders}
			{i18n}
		/>
	{/if}
	{#if view === VIEWS.SIGN_IN}
		{#if !onlyThirdPartyProviders}
			<EmailAuth
				{appearance}
				{supabaseClient}
				bind:authView={view}
				{redirectTo}
				{magicLink}
				{showLinks}
				{i18n}
				{additionalData}
			/>
		{/if}
	{/if}
	{#if view === VIEWS.SIGN_UP}
		{#if !onlyThirdPartyProviders}
			<EmailAuth
				{appearance}
				{supabaseClient}
				bind:authView={view}
				{redirectTo}
				{magicLink}
				{showLinks}
				{additionalData}
				{i18n}><slot /></EmailAuth
			>
		{/if}
	{/if}
	{#if view === VIEWS.FORGOTTEN_PASSWORD}
		<ForgottenPassword {i18n} {supabaseClient} bind:authView={view} {showLinks} {appearance} {redirectTo} />
	{/if}
	{#if view === VIEWS.MAGIC_LINK}
		<MagicLink {i18n} {supabaseClient} bind:authView={view} {appearance} {redirectTo} {showLinks} />
	{/if}
	{#if view === VIEWS.UPDATE_PASSWORD}
		<UpdatePassword {i18n} {supabaseClient} bind:authView={view} {appearance} {showLinks} />
	{/if}
	{#if view === VIEWS.VERIFY_OTP}
		<VerifyOtp {i18n} {supabaseClient} bind:authView={view} {appearance} {showLinks} {otpType} />
	{/if}
</div>

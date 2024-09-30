<script>
    import { goto } from '$app/navigation';
    import { userSignIn, userSignUp, forgotPassword, resetPassword } from '$lib/apis/auths';
    import Spinner from '$lib/components/common/Spinner.svelte';
    import { WEBUI_API_BASE_URL, WEBUI_BASE_URL } from '$lib/constants';
    import { WEBUI_NAME, config, user } from '$lib/stores';
    import { onMount, getContext } from 'svelte';
    import { toast } from 'svelte-sonner';
    import { generateInitialsImage } from '$lib/utils';

    const i18n = getContext('i18n');

    let loaded = false;
    let mode = 'signin';
    let name = '';
    let email = '';
    let password = '';
    let newPassword = '';
    let token = '';

    const setSessionUser = async (sessionUser) => {
        if (sessionUser) {
            toast.success($i18n.t(`You're now logged in.`));
            await user.set(sessionUser);
            localStorage.setItem('token', sessionUser.token); // Store the token
            goto('/');
        }
    };

    const signInHandler = async () => {
        const sessionUser = await userSignIn(email, password).catch((error) => {
            toast.error(error);
            return null;
        });
        await setSessionUser(sessionUser);
    };

    const signUpHandler = async () => {
        const sessionUser = await userSignUp(name, email, password, generateInitialsImage(name)).catch((error) => {
            toast.error(error);
            return null;
        });
        await setSessionUser(sessionUser);
    };

    const forgotPasswordHandler = async () => {
        try {
            await forgotPassword(email);
            toast.success("Reset link sent to your email.");
            mode = 'signin'; // Switch back to sign-in mode after sending the reset link
        } catch (error) {
            toast.error("Failed to send reset link: " + error.message);
        }
    };

    const resetPasswordHandler = async () => {
        try {
            await resetPassword(token, newPassword);
            toast.success("Password successfully reset.");
            mode = 'signin'; // Switch back to sign-in mode after resetting the password
        } catch (error) {
            toast.error("Failed to reset password: " + error.message);
        }
    };

    const submitHandler = async () => {
        if (mode === 'signin') {
            await signInHandler();
        } else if (mode === 'signup') {
            await signUpHandler();
        } else if (mode === 'forgot') {
            await forgotPasswordHandler();
        } else if (mode === 'reset') {
            await resetPasswordHandler();
        }
    };
    
    onMount(() => {
        const url = new URL(window.location.href);
        console.log('Full URL:', url.href);
        const params = new URLSearchParams(url.search);
        const tokenParam = params.get('token');

        if (tokenParam) {
            token = tokenParam;
            mode = 'reset';
        }

        loaded = true;
    });
</script>

<svelte:head>
    <title>{`${$WEBUI_NAME}`}</title>
</svelte:head>

{#if loaded}
    <div class="fixed m-10 z-50">
        <div class="flex space-x-2">
            <div class="self-center">
                <img
                    crossorigin="anonymous"
                    src="{WEBUI_BASE_URL}/static/favicon.png"
                    class="w-8 rounded-full"
                    alt="logo"
                />
            </div>
        </div>
    </div>

    <div class="bg-white dark:bg-gray-950 min-h-screen w-full flex justify-center font-mona">
        <div class="w-full sm:max-w-md px-10 min-h-screen flex flex-col text-center">
            {#if ($config?.trusted_header_auth ?? false) || $config?.auth === false}
                <div class="my-auto pb-10 w-full">
                    <div class="flex items-center justify-center gap-3 text-xl sm:text-2xl text-center font-bold dark:text-gray-200">
                        <div>
                            {$i18n.t('Signing in')}
                            {$i18n.t('to')}
                            {$WEBUI_NAME}
                        </div>
                        <div>
                            <Spinner />
                        </div>
                    </div>
                </div>
            {:else}
                <div class="my-auto pb-10 w-full dark:text-gray-100">
                    <form
                        class="flex flex-col justify-center"
                        on:submit|preventDefault={submitHandler}
                    >
                        <div class="mb-1">
                            <div class="text-2xl font-bold">
                                {mode === 'signin' ? $i18n.t('Sign in') : 
                                 mode === 'signup' ? $i18n.t('Sign up') : 
                                 mode === 'forgot' ? $i18n.t('Forgot Password') : 
                                 $i18n.t('Reset Password')}
                                {$i18n.t('to')}
                                {$WEBUI_NAME}
                            </div>

                            {#if mode === 'signup'}
                                <div class="mt-1 text-xs font-medium text-gray-500">
                                    ⓘ {$WEBUI_NAME}
                                    {$i18n.t('does not make any external connections, and your data stays securely on your locally hosted server.')}
                                </div>
                            {/if}
                        </div>

                        <div class="flex flex-col mt-4">
                            {#if mode === 'signup'}
                                <div>
                                    <div class="text-sm font-semibold text-left mb-1">{$i18n.t('Name')}</div>
                                    <input
                                        id="name"
                                        name="name"
                                        bind:value={name}
                                        type="text"
                                        class="px-5 py-3 rounded-2xl w-full text-sm outline-none border dark:border-none dark:bg-gray-900"
                                        autocomplete="name"
                                        placeholder={$i18n.t('Enter Your Full Name')}
                                        required
                                    />
                                </div>
                                <hr class="my-3 dark:border-gray-900" />
                            {/if}

                            {#if mode !== 'reset'}
                                <div class="mb-2">
                                    <div class="text-sm font-semibold text-left mb-1">{$i18n.t('Email')}</div>
                                    <input
                                        id="email"
                                        name="email"
                                        bind:value={email}
                                        type="email"
                                        class="px-5 py-3 rounded-2xl w-full text-sm outline-none border dark:border-none dark:bg-gray-900"
                                        autocomplete="email"
                                        placeholder={$i18n.t('Enter Your Email')}
                                        required
                                    />
                                </div>
                            {/if}

                            {#if mode === 'signin' || mode === 'signup'}
                                <div>
                                    <div class="text-sm font-semibold text-left mb-1">{$i18n.t('Password')}</div>
                                    <input
                                        id="password"
                                        name="password"
                                        bind:value={password}
                                        type="password"
                                        class="px-5 py-3 rounded-2xl w-full text-sm outline-none border dark:border-none dark:bg-gray-900"
                                        placeholder={$i18n.t('Enter Your Password')}
                                        autocomplete="current-password"
                                        required
                                    />
                                </div>
                            {/if}

                            {#if mode === 'reset'}
                                <div>
                                    <div class="text-sm font-semibold text-left mb-1">{$i18n.t('New Password')}</div>
                                    <input
                                        id="newPassword"
                                        name="newPassword"
                                        bind:value={newPassword}
                                        type="password"
                                        class="px-5 py-3 rounded-2xl w-full text-sm outline-none border dark:border-none dark:bg-gray-900"
                                        placeholder={$i18n.t('Enter Your New Password')}
                                        autocomplete="new-password"
                                        required
                                    />
                                </div>
                            {/if}
                        </div>

                        {#if mode === 'signin'}
                            <div class="mt-4 text-sm text-center">
                                <a href="#" class="font-medium underline" on:click={() => mode = 'forgot'}>{$i18n.t('Forgot Password?')}</a>
                            </div>
                        {/if}

                        <div class="mt-5">
                            <button
                                class="bg-gray-900 hover:bg-gray-800 w-full rounded-2xl text-white font-semibold text-sm py-3 transition"
                                type="submit"
                            >
                                {mode === 'signin' ? $i18n.t('Sign in') :
                                mode === 'signup' ? $i18n.t('Create Account') :
                                mode === 'forgot' ? $i18n.t('Send Reset Link') :
                                $i18n.t('Reset Password')}
                            </button>

                            <div class="mt-4 text-sm text-center">
                                {#if mode === 'signin'}
                                    {$i18n.t("Don't have an account?")}
                                    <button
                                        class="font-medium underline"
                                        type="button"
                                        on:click={() => mode = 'signup'}
                                    >
                                        {$i18n.t('Sign up')}
                                    </button>
                                {:else if mode === 'signup'}
                                    {$i18n.t('Already have an account?')}
                                    <button
                                        class="font-medium underline"
                                        type="button"
                                        on:click={() => mode = 'signin'}
                                    >
                                        {$i18n.t('Sign in')}
                                    </button>
                                {:else if mode === 'forgot'}
                                    {$i18n.t('Remembered your password?')}
                                    <button
                                        class="font-medium underline"
                                        type="button"
                                        on:click={() => mode = 'signin'}
                                    >
                                        {$i18n.t('Sign in')}
                                    </button>
                                {/if}
                            </div>
                        </div>
                    </form>
                </div>
            {/if}
        </div>
    </div>
{/if}

<style>
    .font-mona {
        font-family: 'Mona Sans', -apple-system, 'Arimo', ui-sans-serif, system-ui, 'Segoe UI', Roboto,
            Ubuntu, Cantarell, 'Noto Sans', sans-serif, 'Helvetica Neue', Arial, 'Apple Color Emoji',
            'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
    }
</style>

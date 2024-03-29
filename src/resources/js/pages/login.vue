<script setup>
import { onMounted, ref } from 'vue'
import authService from '../services/authService'
import { useToast } from "vue-toastification"
import handleApiError from '../libs/handleApiError'
import { useRouter } from 'vue-router'
import { LOCALSTORAGE_KEY } from '../libs/constants'

const router = useRouter()
const errors = ref([])
const errLogin = ref("")
const email = ref("")
const password = ref("")
const loading = ref(true)
const loadingForm = ref(false)

async function login({ email, password }) {
    try {
        loadingForm.value = true
        const res = await authService.login({ email, password })
        const { access_token, refresh_token, user } = res.data.data
        const toast = useToast()
        toast.success(`Login success Selamat datang ${user.name}`, {
            timeout: 2000
        })
        localStorage.setItem(LOCALSTORAGE_KEY.ACCESS_TOKEN, access_token)
        localStorage.setItem(LOCALSTORAGE_KEY.REFRESH_TOKEN, refresh_token)
        router.push('/')
    } catch (err) {
        loadingForm.value = false
        handleApiError(err, router)
        if (err.response) {
            errLogin.value = err.response?.data?.msg
        }
    }
}

async function submit(e) {
    e.preventDefault()

    try {
        let valueErrors = errors.value

        if (!email.value) {
            valueErrors.push('email required')
        }
        if (!email.value) {
            valueErrors.push('password required')
        }

        if (valueErrors.length > 0) {
            errors.value = valueErrors
        } else {
            await login({ email: email.value, password: password.value })
        }
    } catch (err) {
        // err
    }
}

onMounted(function () {
    const accessToken = localStorage.getItem('access_token')
    const refreshToken = localStorage.getItem('refresh_token')
    if (accessToken && refreshToken) {
        router.push('/')
    }
    loading.value = false
})
</script>

<template>
    <section v-if="!loading" class="bg-gray-50 dark:bg-gray-900">
        <div class="flex flex-col items-center justify-center mx-auto md:h-screen lg:py-0">
            <a href="#" class="flex items-center mb-6 text-2xl font-semibold text-gray-900 dark:text-white">
                <img class="w-32" src="https://nabitu.id/img/nabitu-logo/nabitu_logo-with-text.png" alt="logo">    
            </a>
            <div class="w-full bg-white rounded-lg shadow dark:border md:mt-0 sm:max-w-md xl:p-0 dark:bg-gray-800 dark:border-gray-700">
                <div class="p-6 space-y-4 md:space-y-6 sm:p-8">
                    <h1 class="text-xl font-bold leading-tight tracking-tight text-gray-900 md:text-2xl dark:text-white">
                        Sign in to your account
                    </h1>
                    <form class="space-y-4 md:space-y-6" @submit="submit">
                        <div>
                            <label for="email" class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">Your email</label>
                            <input v-model="email" type="email" name="email" id="email" class="bg-gray-50 border border-gray-300 text-gray-900 sm:text-sm rounded-lg focus:ring-primary-600 focus:border-primary-600 block w-full p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500" placeholder="name@company.com">
                        </div>
                        <div>
                            <label for="password" class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">Password</label>
                            <input v-model="password" type="password" name="password" id="password" placeholder="••••••••" class="bg-gray-50 border border-gray-300 text-gray-900 sm:text-sm rounded-lg focus:ring-primary-600 focus:border-primary-600 block w-full p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500">
                        </div>
                        <ul v-if="errors.length > 0" class="max-w-md space-y-1 text-gray-500 list-disc list-inside dark:text-gray-400">
                            <li class="mt-2 text-xs text-red-600 dark:text-red-400" v-for="(item) of errors"><span class="font-medium">Error!</span> {{ item }}</li>
                        </ul>
                        <div class="text-xs text-red-600" data-error="errlogin">{{ errLogin }}</div>
                        <!-- <button type="submit" class="w-full text-white bg-primary-600 hover:bg-primary-700 focus:ring-4 focus:outline-none focus:ring-primary-300 font-medium rounded-lg text-sm px-5 py-2.5 text-center dark:bg-primary-600 dark:hover:bg-primary-700 dark:focus:ring-primary-800">Sign in</button> -->
                        <button :disabled="loadingForm" type="submit" class="flex items-center justify-between w-full text-white bg-primary-600 hover:bg-primary-700 focus:ring-4 focus:outline-none focus:ring-primary-300 font-medium rounded-lg text-sm px-5 py-2.5 text-center dark:bg-primary-600 dark:hover:bg-primary-700 dark:focus:ring-primary-800">
                            <div /><div class="flex items-center"><v-icon v-if="loading" name="wi-refresh-alt" scale="2" class="animate-spin" /> <div>Sign in</div></div><div />
                        </button>
                    </form>
                </div>
            </div>
        </div>
    </section>
</template>

<style scoped></style>
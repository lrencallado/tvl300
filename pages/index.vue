<script setup lang="ts">
import { Listbox, ListboxButton, ListboxLabel, ListboxOption, ListboxOptions } from '@headlessui/vue'
import { CheckIcon, ChevronUpDownIcon } from '@heroicons/vue/20/solid'
import { MagnifyingGlassIcon } from "@heroicons/vue/20/solid"; 
import { format, parseISO } from 'date-fns'

interface Domain {
    domain_name: string,
    registrar_name: string,
    registration_date: any,
    estimated_domain_age: number,
    hostnames: string[]
}

interface Contact {
    registrant_name: string,
    technical_contact_name: string,
    administrative_contact_name: string,
    contact_email: string
}

const infoTypes = [
    {
        label: 'Domain Information',
        value: 'domain'
    },
    {
        label: 'Contact Information',
        value: 'contact'
    }
]

const isErrorAlert = ref(false)
const alertMessage = ref('')
const selectedType = ref(infoTypes[0])
const domainName = ref('')
const processing = ref(false)
const config = useRuntimeConfig()

let domainInformation = ref<Domain>({
    domain_name: '',
    registrar_name: '',
    registration_date: '',
    estimated_domain_age: 0,
    hostnames: []
})

let contactInformation = ref<Contact>({
    registrant_name: '',
    technical_contact_name: '',
    administrative_contact_name: '',
    contact_email: ''
})

const hideErrorAlert = () => {
    isErrorAlert.value = false
    alertMessage.value = ''
};

const showNotification = () => {
    isErrorAlert.value = true;
    setTimeout(() => {
        hideErrorAlert()
    }, 3000); // Hide notification after 3 seconds
};

const domainLookup = async() => {
    try {
        processing.value = true

        // Perform the fetch request to the external API
        const response = await $fetch <Record<string, any>>(`${config.public.apiBaseURL}/api/v1/domain?domain_name=${domainName.value}&type=${selectedType.value.value}`, {
            headers: {
                'Accept': 'application/json',
                //'Authorization': `Bearer `,
            }
        });

        if (selectedType.value.value == 'domain') {
            domainInformation.value = response.data
        } else {
            contactInformation.value = response.data
        }

        processing.value = false
    } catch (error: any) {
        const errorMessage = error.response._data?.message || error.response.statusText || 'An error occurred'
        alertMessage.value = errorMessage
        showNotification()
        processing.value = false
    }
}

function formatDate(date: any) {
    // Parse the date string into a JavaScript Date object
    const registrationDate = parseISO(date);

    return format(registrationDate, 'MMMM d, yyyy h:mm a')
}

function formatHostnames(hostnames: string[]): string {
    // Join array elements into a comma-separated string
    let result = hostnames.join(', ');
    
    // Truncate if length exceeds 25 characters
    if (result.length > 25) {
        result = result.substring(0, 25) + '...';
    }
    
    return result;
}
</script>

<template>
    <div class="sm:p-20 p-5 bg-gray-50 min-h-screen dark:bg-gray-900 font-sans antialiased">
        <div class="flex mb-6 space-x-4">
            <div>
                <input v-model="domainName" type="text" placeholder="google.com" class="block w-full rounded-md border-0 py-1.5 px-2 text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-indigo-500 sm:text-sm sm:leading-6 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white" />
            </div>
            <div>
                <Listbox as="div" v-model="selectedType">
                    <!-- <ListboxLabel class="block text-sm font-medium leading-6 text-gray-900">Assigned to</ListboxLabel> -->
                    <div class="relative">
                    <ListboxButton class="relative w-full cursor-default rounded-md bg-white py-1.5 pl-3 pr-10 text-left text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 focus:outline-none focus:ring-2 focus:ring-indigo-500 sm:text-sm sm:leading-6 dark:bg-gray-700 dark:placeholder-gray-400 dark:text-white">
                        <span class="flex items-center">
                        <span class="block truncate">{{ selectedType.label }}</span>
                        </span>
                        <span class="pointer-events-none absolute inset-y-0 right-0 ml-3 flex items-center pr-2">
                        <ChevronUpDownIcon class="h-5 w-5 text-gray-400" aria-hidden="true" />
                        </span>
                    </ListboxButton>

                    <transition leave-active-class="transition ease-in duration-100" leave-from-class="opacity-100" leave-to-class="opacity-0">
                        <ListboxOptions class="absolute z-10 mt-1 max-h-56 w-full overflow-auto rounded-md bg-white py-1 text-base shadow-lg ring-1 ring-black ring-opacity-5 focus:outline-none sm:text-sm dark:bg-gray-700 dark:text-white">
                        <ListboxOption as="template" v-for="infoType in infoTypes" :key="infoType.value" :value="infoType" v-slot="{ active, selectedType }">
                            <li :class="[active ? 'bg-indigo-600 text-white' : 'text-gray-900 dark:text-gray-400', 'relative cursor-default select-none py-2 pl-3 pr-9']">
                                <div class="flex items-center">
                                    <span :class="[selectedType ? 'font-semibold' : 'font-normal', 'block truncate']">{{ infoType.label }}</span>
                                </div>

                                <span v-if="selectedType == infoType" :class="[active ? 'text-white' : 'text-indigo-600', 'absolute inset-y-0 right-0 flex items-center pr-4']">
                                    <CheckIcon class="h-5 w-5" aria-hidden="true" />
                                </span>
                            </li>
                        </ListboxOption>
                        </ListboxOptions>
                    </transition>
                    </div>
                </Listbox>
            </div>
            <div>
                <button :disabled="processing" @click="domainLookup" class="disabled:bg-indigo-600/50 flex rounded-md bg-indigo-600 px-3 py-2 text-sm font-semibold text-white shadow-sm hover:bg-indigo-500 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600">
                    <MagnifyingGlassIcon v-if="!processing" class="h-5 w-5 text-white me-2" />
                    <div v-else role="status">
                        <svg aria-hidden="true" class="w-5 h-5 me-2 text-gray-200 animate-spin dark:text-gray-600 fill-blue-600" viewBox="0 0 100 101" fill="none" xmlns="http://www.w3.org/2000/svg">
                            <path d="M100 50.5908C100 78.2051 77.6142 100.591 50 100.591C22.3858 100.591 0 78.2051 0 50.5908C0 22.9766 22.3858 0.59082 50 0.59082C77.6142 0.59082 100 22.9766 100 50.5908ZM9.08144 50.5908C9.08144 73.1895 27.4013 91.5094 50 91.5094C72.5987 91.5094 90.9186 73.1895 90.9186 50.5908C90.9186 27.9921 72.5987 9.67226 50 9.67226C27.4013 9.67226 9.08144 27.9921 9.08144 50.5908Z" fill="currentColor"/>
                            <path d="M93.9676 39.0409C96.393 38.4038 97.8624 35.9116 97.0079 33.5539C95.2932 28.8227 92.871 24.3692 89.8167 20.348C85.8452 15.1192 80.8826 10.7238 75.2124 7.41289C69.5422 4.10194 63.2754 1.94025 56.7698 1.05124C51.7666 0.367541 46.6976 0.446843 41.7345 1.27873C39.2613 1.69328 37.813 4.19778 38.4501 6.62326C39.0873 9.04874 41.5694 10.4717 44.0505 10.1071C47.8511 9.54855 51.7191 9.52689 55.5402 10.0491C60.8642 10.7766 65.9928 12.5457 70.6331 15.2552C75.2735 17.9648 79.3347 21.5619 82.5849 25.841C84.9175 28.9121 86.7997 32.2913 88.1811 35.8758C89.083 38.2158 91.5421 39.6781 93.9676 39.0409Z" fill="currentFill"/>
                        </svg>
                        <span class="sr-only">Loading...</span>
                    </div>
                    Lookup
                </button>
            </div>
        </div>
        <div class="bg-white px-5 py-5 rounded-md shadow-sm mb-6 dark:bg-gray-800 dark:border-gray-700">
            <div class="px-4 sm:px-0 mb-4">
                <h3 class="text-base font-semibold leading-7 text-gray-900 dark:text-white">Domain Information</h3>
            </div>
            <div class="relative overflow-x-auto">
                <table class="w-full text-sm text-left rtl:text-right text-gray-500 dark:text-gray-400">
                    <thead class="text-xs text-gray-900 uppercase bg-gray-50 dark:bg-gray-700 dark:text-gray-400">
                        <tr>
                            <th scope="col" class="px-6 py-3">
                                Domain Name
                            </th>
                            <th scope="col" class="px-6 py-3">
                                Registrar
                            </th>
                            <th scope="col" class="px-6 py-3">
                                Registration Date
                            </th>
                            <th scope="col" class="px-6 py-3">
                                Estimated Domain Age
                            </th>
                            <th scope="col" class="px-6 py-3">
                                Hostnames
                            </th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-if="domainInformation.domain_name" class="bg-white border-b dark:bg-gray-800 dark:border-gray-700">
                            <th scope="row" class="px-6 py-4 font-medium text-gray-900 whitespace-nowrap dark:text-white">
                                {{ domainInformation.domain_name }}
                            </th>
                            <td class="px-6 py-4">
                                {{ domainInformation.registrar_name }}
                            </td>
                            <td class="px-6 py-4">
                                {{ domainInformation.registration_date ? formatDate(domainInformation.registration_date) : '' }}
                            </td>
                            <td class="px-6 py-4">
                                {{ domainInformation.estimated_domain_age ? domainInformation.estimated_domain_age + ' days' : '' }}
                            </td>
                            <td class="px-6 py-4">
                                {{ domainInformation.hostnames ? formatHostnames(domainInformation.hostnames) : '' }}
                            </td>
                        </tr>
                        <tr v-else class="bg-white border-b dark:bg-gray-800 dark:border-gray-700">
                            <td class="px-6 py-4"  colspan="5">
                                No data.
                            </td>   
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>

        <div class="bg-white px-5 py-5 rounded-md shadow-sm dark:bg-gray-800 dark:border-gray-700">
            <div class="px-4 sm:px-0 mb-4">
                <h3 class="text-base font-semibold leading-7 text-gray-900 dark:text-white">Contact Information</h3>
            </div>
            <div class="relative overflow-x-auto">
                <table class="w-full text-sm text-left rtl:text-right text-gray-500 dark:text-gray-400">
                    <thead class="text-xs text-gray-900 uppercase bg-gray-50 dark:bg-gray-700 dark:text-gray-400">
                        <tr>
                            <th scope="col" class="px-6 py-3">
                                Registrant Name
                            </th>
                            <th scope="col" class="px-6 py-3">
                                Technical Contact Name
                            </th>
                            <th scope="col" class="px-6 py-3">
                                Administrative Contact Name
                            </th>
                            <th scope="col" class="px-6 py-3">
                                Contact Email
                            </th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-if="contactInformation.contact_email" class="bg-white border-b dark:bg-gray-800 dark:border-gray-700">
                            <th scope="row" class="px-6 py-4 font-medium text-gray-900 whitespace-nowrap dark:text-white">
                                {{ contactInformation.registrant_name }}
                            </th>
                            <td class="px-6 py-4">
                                {{ contactInformation.technical_contact_name }}
                            </td>
                            <td class="px-6 py-4">
                                {{ contactInformation.administrative_contact_name}}
                            </td>
                            <td class="px-6 py-4">
                                {{ contactInformation.contact_email }}
                            </td>
                        </tr>
                        <tr v-else class="bg-white border-b dark:bg-gray-800 dark:border-gray-700">
                            <td class="px-6 py-4" colspan="4">
                                No data.
                            </td>   
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>
    </div>

    <transition name="fade">
        <div
        v-if="isErrorAlert"
        class="fixed top-0 right-4 text-white p-4"
        role="alert"
        >
            <div id="alert-2" class="flex items-center p-4 mb-4 text-red-800 rounded-lg bg-red-50 dark:bg-gray-800 dark:text-red-400" role="alert">
                <svg class="flex-shrink-0 w-4 h-4" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="currentColor" viewBox="0 0 20 20">
                    <path d="M10 .5a9.5 9.5 0 1 0 9.5 9.5A9.51 9.51 0 0 0 10 .5ZM9.5 4a1.5 1.5 0 1 1 0 3 1.5 1.5 0 0 1 0-3ZM12 15H8a1 1 0 0 1 0-2h1v-3H8a1 1 0 0 1 0-2h2a1 1 0 0 1 1 1v4h1a1 1 0 0 1 0 2Z"/>
                </svg>
                <span class="sr-only">Info</span>
                <div class="ms-3 text-sm font-medium">
                    {{ alertMessage }}
                </div>
                <button @click="hideErrorAlert" type="button" class="ms-auto -mx-1.5 -my-1.5 bg-red-50 text-red-500 rounded-lg focus:ring-2 focus:ring-red-400 p-1.5 hover:bg-red-200 inline-flex items-center justify-center h-8 w-8 dark:bg-gray-800 dark:text-red-400 dark:hover:bg-gray-700" data-dismiss-target="#alert-2" aria-label="Close">
                    <span class="sr-only">Close</span>
                    <svg class="w-3 h-3" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 14 14">
                    <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="m1 1 6 6m0 0 6 6M7 7l6-6M7 7l-6 6"/>
                    </svg>
                </button>
            </div>
        </div>
    </transition>
</template>

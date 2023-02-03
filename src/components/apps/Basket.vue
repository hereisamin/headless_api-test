<template>
    <div v-if="!appStarted" class="p-5 flex justify-center items-center w-full">
        <div class="px-5">
            <label class="flex items-center relative w-max cursor-pointer select-none">
            <span class="text-lg font-bold mr-3 text-blue-900">Break Mode</span>
            <input type="checkbox" v-model="breakMode" class="appearance-none transition-colors cursor-pointer w-14 h-7 rounded-full focus:outline-none ring-2 ring-blue-500 bg-red-500" />
            <span class="absolute font-medium text-xs uppercase right-1 text-white"> OFF </span>
            <span class="absolute font-medium text-xs uppercase right-8 text-white"> ON </span>
            <span class="w-7 h-7 right-7 absolute rounded-full transform transition-transform bg-gray-200" />
            </label>  
        </div>
        <button class="px-6 py-1 text-lg text-blue-900 bg-blue-100 shadow-sm font-bold rounded-md border " @click="sendRequests">Start</button>
    </div>
    <template v-if="appStarted">
        <CheckList v-for="request in sendingRequestArray" :key="request.name"
        :request="request"
        :loading="request.loading"
        :error="request.error"
        :request-name="request.break ? `${request.name} [${request.break}]` : request.name"
        :method="request.values.method"
        :result="request.result" />
    </template>
</template>

<script setup>
import CheckList from '../CheckList.vue'
import axios from 'axios'
import { computed, reactive, ref} from 'vue'

const props = defineProps({
    token: {
        type: Object,
        default: () => {},
    },
})
const baseUrl = 'http://192.168.1.117:5079'
const breakMode = ref(false)
const url = function (extend) { return {url: `${baseUrl}/api/v1${extend}`} }
const basketId = ref('')
const cultureCode = 'en-US'
const currency = 'EUR'
const newToken = ref({})
const accessToken = computed(() => newToken.value.access_token ? 'Bearer ' + newToken.value.access_token : 'Bearer ' + props.token.access)
const countryId = ref('')
const paymentMethodId = ref('')
const priceGroupId = ref('')
const catalogId = ref('')
const shippingMethodId = ref('')
const lineId = ref('')
const basketCustomPropertyId = ref('')
const orderLineCustomPropertyId = ref('')
const fakeGuid = '876a0ff1-0700-40ac-b006-0c9758f234a4'

const sendingRequestArray = computed(() => breakMode.value ? requestArray : requestArray.filter(request => !request.break))

const requestArray = reactive([
    {
        name: 'Refresh Token',
        values: computed(() => {
            return {
                ...url('/oauth/token'),
                method: 'POST',
                data: {
                    'grant_type' : 'refresh_token',
                    'refresh_token' : newToken.value.refresh_token ? newToken.value.refresh_token : props.token.refresh,
                },
                headers: {
                'content-type': 'application/x-www-form-urlencoded',
                },
                auth: {
                    username: props.token.client.id,
                    password: props.token.client.secret,
                },
            }
        }),
        success: (response) => {
            console.log('Refresh Token :', response)
        },
    },{
        break: 'currency',
        name: 'Create Basket',
        values: computed(() => {
            return {
                ...url('/baskets'),
                method: 'POST',
                data: {
                    cultureCode,
                    currency: 'sss',
                },
                headers: {
                    Authorization: accessToken.value,
                }
            }
        }),
        success: (response) => {
            console.log('BREAK - Create Basket :', response)
        },
    },
    {
        name: 'Create Basket',
        values: computed(() => {
            return {
                ...url('/baskets'),
                method: 'POST',
                data: {
                    cultureCode,
                    currency,
                },
                headers: {
                    Authorization: accessToken.value,
                }
            }
        }),
        success: (response) => {
            basketId.value = response.data.basketId // '1127af1d-32a5-6dda-bde4-500a351ff854'
            console.log('Create Basket :', response)
        },
    },
    {
        break: 'basket id',
        name: 'Basket',
        values: computed(() => {
            return {
                ...url(`/baskets/1127af1d-32a5-6dda-bde4-500a351ff854`),
                method: 'GET',
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
        success: (response) => {
            console.log('Basket :', response)
        },
    },
    {
        name: 'Basket',
        values: computed(() => {
            return {
                ...url(`/baskets/${basketId.value}`),
                method: 'GET',
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
        success: (response) => {
            console.log('Basket :', response)
        },
    },
    {
        name: 'Price Groups',
        values: computed(() => {
            return {
                ...url('/price-groups'),
                method: 'GET',
                params:{
                    cultureCode,
                },
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
        success: (response) => {
            priceGroupId.value = response.data.priceGroups[0].id
            console.log('Price Group :', response)
        },
    },
    {
        name: 'Countries',
        values: computed(() => {
            return {
                ...url('/countries'),
                method: 'GET',
                headers: {
                    Authorization: accessToken.value,
                },      
            }
        }),
        success: (response) => {
            countryId.value = response.data.countries[0].id
            console.log('Countries :', response)
        },
    },
    {
        break: 'countryId',
        name: 'Payment Methods',
        values: computed(() => {
            return {
                ...url('/payment-methods'),
                method: 'GET',
                params:{
                    cultureCode,
                    countryId: '1527af1d-32a5-6dda-bde4-500a351ff867',
                },
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
        success: (response) => {
            console.log('BREAK - Payment Methods :', response)
        },
    },
    {
        name: 'Payment Methods',
        values: computed(() => {
            return {
                ...url('/payment-methods'),
                method: 'GET',
                params:{
                    cultureCode,
                    countryId: countryId.value,
                },
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
        success: (response) => {
            paymentMethodId.value = response.data.paymentMethods[0].id
            console.log('Payment Methods :', response)
        },
    },
    {
        break: 'priceGroupId - Valid guid',
        name: 'Catalogs',
        values: computed(() => {
            return {
                ...url('/catalogs'),
                method: 'GET',
                headers: {
                    Authorization: accessToken.value,
                },
                params:{
                    cultureCode,
                    priceGroupId: '1127af1d-32a5-6dda-bde4-500a351ff854',
                },
            }
        }),
        success: (response) => {
            console.log('BREAK - Catalogs :', response)
        },
    },
    {
        break: 'priceGroupId - Not valid guid',
        name: 'Catalogs',
        values: computed(() => {
            return {
                ...url('/catalogs'),
                method: 'GET',
                headers: {
                    Authorization: accessToken.value,
                },
                params:{
                    cultureCode,
                    priceGroupId: '1127af1d-32a5-6dda-bde4',
                },
            }
        }),
        success: (response) => {
            console.log('BREAK - Catalogs :', response)
        },
    },
    {
        name: 'Catalogs',
        values: computed(() => {
            return {
                ...url('/catalogs'),
                method: 'GET',
                headers: {
                    Authorization: accessToken.value,
                },
                params:{
                    cultureCode,
                    priceGroupId: priceGroupId.value, 
                },
            }
        }),
        success: (response) => {
            catalogId.value = response.data.catalogs[0].id
            console.log('Catalogs :', response)
        },
    },
    {
        break:'countryId - valid guid',
        name: 'Shipping Methods',
        values: computed(() => {
            return {
                ...url('/shipping-methods'),
                method: 'GET',
                params:{
                    cultureCode,
                    countryId: '1127af1d-32a5-6dda-bde4-500a351ff854',
                    priceGroupId: priceGroupId.value,
                },
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
        success: (response) => {
            console.log('Payment Methods :', response)
        },
    },
    {
        break:'priceGroupId - valid guid',
        name: 'Shipping Methods',
        values: computed(() => {
            return {
                ...url('/shipping-methods'),
                method: 'GET',
                params:{
                    cultureCode,
                    countryId: countryId.value,
                    priceGroupId: '1127af1d-32a5-6dda-bde4-500a351ff854',
                },
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
        success: (response) => {
            console.log('Payment Methods :', response)
        },
    },
    {
        name: 'shipping Methods',
        values: computed(() => {
            return {
                ...url('/shipping-methods'),
                method: 'GET',
                params:{
                    cultureCode,
                    countryId: countryId.value,
                    priceGroupId: priceGroupId.value,
                },
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
        success: (response) => {
            shippingMethodId.value = response.data.shippingMethods[0].id
            console.log('Payment Methods :', response)
        },
    },
    {
        break: 'PriceGroupId',
        name: 'Adding a line item to a basket',
        values: computed(() => {
            return {
                ...url(`/baskets/${ basketId.value }/lines`),
                method: 'POST',
                data: {
                    Quantity: 1,
                    Sku: '200-000-001',
                    VariantSku: '002',
                    PriceGroupId: '878b0ff1-0700-40ac-b006-0e9858f234a5',
                    ProductCatalogId: catalogId.value,
                    cultureCode,
                },
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
        success: (response) => {
            console.log('Adding a line item to a basket :', response)
        },
    },
    {
        break: 'ProductCatalogId',
        name: 'Adding a line item to a basket',
        values: computed(() => {
            return {
                ...url(`/baskets/${ basketId.value }/lines`),
                method: 'POST',
                data: {
                    Quantity: 1,
                    Sku: '200-000-001',
                    VariantSku: '002',
                    PriceGroupId: priceGroupId.value,
                    ProductCatalogId: '878b0ff1-0800-40ac-b006-0e9858f234a4',
                    cultureCode,
                },
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
        success: (response) => {
            console.log('Adding a line item to a basket :', response)
        },
    },
    {
        break: 'sku',
        name: 'Adding a line item to a basket',
        values: computed(() => {
            return {
                ...url(`/baskets/${ basketId.value }/lines`),
                method: 'POST',
                data: {
                    Quantity: 1,
                    Sku: 'notExistingSKU',
                    VariantSku: '002',
                    PriceGroupId: priceGroupId.value, //'878b0ff1-0800-40ac-b006-0e9858f234a5'
                    ProductCatalogId: catalogId.value,
                    cultureCode,
                },
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
        success: (response) => {
            console.log('Adding a line item to a basket :', response)
        },
    },
    {
        break: 'VariantSku',
        name: 'Adding a line item to a basket',
        values: computed(() => {
            return {
                ...url(`/baskets/${ basketId.value }/lines`),
                method: 'POST',
                data: {
                    Quantity: 1,
                    Sku: '200-000-001',
                    VariantSku: 'notExistingVariantSKU',
                    PriceGroupId: priceGroupId.value, //'878b0ff1-0800-40ac-b006-0e9858f234a5'
                    ProductCatalogId: catalogId.value,
                    cultureCode,
                },
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
        success: (response) => {
            console.log('Adding a line item to a basket :', response)
        },
    },
    {
        break: 'quantity -1',
        name: 'Adding a line item to a basket',
        values: computed(() => {
            return {
                ...url(`/baskets/${ basketId.value }/lines`),
                method: 'POST',
                data: {
                    quantity: -1,
                    sku: '200-000-001',
                    variantSku: '002',
                    priceGroupId: priceGroupId.value, //'878b0ff1-0800-40ac-b006-0e9858f234a5'
                    productCatalogId: catalogId.value,
                    cultureCode,
                },
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
        success: (response) => {
            console.log('Adding a line item to a basket :', response)
        },
    },
    {
        name: 'Adding a line item to a basket',
        values: computed(() => {
            return {
                ...url(`/baskets/${ basketId.value }/lines`),
                method: 'POST',
                data: {
                    quantity: 1,
                    sku: '200-000-001',
                    variantSku: '002',
                    priceGroupId: priceGroupId.value, //'878b0ff1-0800-40ac-b006-0e9858f234a5'
                    productCatalogId: catalogId.value,
                    cultureCode,
                },
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
        success: (response) => {
            console.log('Adding a line item to a basket :', response)
        },
    },
    {
        break: 'priceGroupId',
        name: 'Add shipping Information',
        values: computed(() => {
            return {
                ...url(`/baskets/${ basketId.value }/shipping`),
                method: 'POST',
                data: {
                    cultureCode,
                    priceGroupId: '878b0ff1-0800-40ac-b005-0e9858f234a5',
                    shippingMethodId: shippingMethodId.value,
                    shippingAddress: {
                        city: 'Aarhus',
                        countryId: countryId.value,
                        email: 'support@ucommerce.dk',
                        firstName: 'John',
                        lastName: 'Doe',
                        line1: 'Klostergade 26',
                        mobileNumber: '88 88 88 88',
                        postalCode: '8000',
                        state: 'Jutland'
                    },
                },
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
    },
    {
        break: 'shippingMethodId',
        name: 'Add shipping Information',
        values: computed(() => {
            return {
                ...url(`/baskets/${ basketId.value }/shipping`),
                method: 'POST',
                data: {
                    cultureCode,
                    priceGroupId: priceGroupId.value,
                    shippingMethodId: '3e77be5c-bcee-4688-94a8-a02987f218f3',
                    shippingAddress: {
                        city: 'Aarhus',
                        countryId: countryId.value,
                        email: 'support@ucommerce.dk',
                        firstName: 'John',
                        lastName: 'Doe',
                        line1: 'Klostergade 26',
                        mobileNumber: '88 88 88 88',
                        postalCode: '8000',
                        state: 'Jutland'
                    },
                },
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
    },
    {
        break: 'countryId',
        name: 'Add shipping Information',
        values: computed(() => {
            return {
                ...url(`/baskets/${ basketId.value }/shipping`),
                method: 'POST',
                data: {
                    cultureCode,
                    priceGroupId: priceGroupId.value,
                    shippingMethodId: shippingMethodId.value,
                    shippingAddress: {
                        city: 'Aarhus',
                        countryId: '4f0c3749-e875-46cc-bbf3-f29265b8a8cf',
                        email: 'support@ucommerce.dk',
                        firstName: 'John',
                        lastName: 'Doe',
                        line1: 'Klostergade 26',
                        mobileNumber: '88 88 88 88',
                        postalCode: '8000',
                        state: 'Jutland'
                    },
                },
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
    },
    {
        name: 'Add shipping Information',
        values: computed(() => {
            return {
                ...url(`/baskets/${ basketId.value }/shipping`),
                method: 'POST',
                data: {
                    cultureCode,
                    priceGroupId: priceGroupId.value,
                    shippingMethodId: shippingMethodId.value,
                    shippingAddress: {
                        city: 'Aarhus',
                        countryId: countryId.value,
                        email: 'support@ucommerce.dk',
                        firstName: 'John',
                        lastName: 'Doe',
                        line1: 'Klostergade 26',
                        mobileNumber: '88 88 88 88',
                        postalCode: '8000',
                        state: 'Jutland'
                    },
                },
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
        success: (response) => {
            console.log('Add shipping Information :', response)
        },
    },

    // Add billing address
    {
        break: 'basketId',
        name: 'Add billing address',
        values: computed(() => {
            return {
                ...url(`/baskets/'4f0c3749-e875-46cc-bbf3-f29265b8a8cf'/billing-address`),
                method: 'POST',
                data: {
                    City: 'Aalborg',
                    firstName: 'John',
                    lastName: 'Doe',
                    postalCode: '9000',
                    line1: 'Klostergade 65',
                    countryId: countryId.value,
                    email:'test@headless.bg',
                    state:'javascript',
                    mobileNumber:'4519194393',
                    attention:'Warning, warning',
                    companyName:'Vrooms',
                },
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
    },
    {
        break: 'countryId',
        name: 'Add billing address',
        values: computed(() => {
            return {
                ...url(`/baskets/${ basketId.value }/billing-address`),
                method: 'POST',
                data: {
                    City: 'Aalborg',
                    firstName: 'John',
                    lastName: 'Doe',
                    postalCode: '9000',
                    line1: 'Klostergade 65',
                    countryId: '4f0c3749-e875-46cc-bbf3-f29265b8a7cf',
                    email:'test@headless.bg',
                    state:'javascript',
                    mobileNumber:'4519194393',
                    attention:'Warning, warning',
                    companyName:'Vrooms',
                },
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
    },
    {
        name: 'Add billing address',
        values: computed(() => {
            return {
                ...url(`/baskets/${ basketId.value }/billing-address`),
                method: 'POST',
                data: {
                    City: 'Aalborg',
                    firstName: 'John',
                    lastName: 'Doe',
                    postalCode: '9000',
                    line1: 'Klostergade 65',
                    countryId: countryId.value,
                    email:'test@headless.bg',
                    state:'javascript',
                    mobileNumber:'4519194393',
                    attention:'Warning, warning',
                    companyName:'Vrooms',
                },
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
        success: (response) => {
            console.log('Add billing address :', response)
        },
    },

    // T Converting a basket to an order
    {
        break: 'basketId',
        name: 'Converting a basket to an order',
        values: computed(() => {
            return {
                ...url(`/payments`),
                method: 'POST',
                data: {
                    basketId: '878b0ff1-0800-40ac-b006-0e9858f244a5',
                    paymentMethodId: paymentMethodId.value,
                    priceGroupId: priceGroupId.value,
                    cultureCode,
                },
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
    },
    {
        break: 'paymentMethodId',
        name: 'Converting a basket to an order',
        values: computed(() => {
            return {
                ...url(`/payments`),
                method: 'POST',
                data: {
                    basketId: basketId.value,
                    paymentMethodId: '878b0ff1-0800-40ac-b006-0e9858f244a5',
                    priceGroupId: priceGroupId.value,
                    cultureCode,
                },
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
    },
    {
        break: 'priceGroupId',
        name: 'Converting a basket to an order',
        values: computed(() => {
            return {
                ...url(`/payments`),
                method: 'POST',
                data: {
                    basketId: basketId.value,
                    paymentMethodId: paymentMethodId.value,
                    priceGroupId: '878b0ff1-0800-40ac-b006-0e9858f244a5',
                    cultureCode,
                },
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
    },
    {
        name: 'Converting a basket to an order',
        values: computed(() => {
            return {
                ...url(`/payments`),
                method: 'POST',
                data: {
                    basketId: basketId.value,
                    paymentMethodId: paymentMethodId.value,
                    priceGroupId: priceGroupId.value,
                    cultureCode,
                },
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
        success: (response) => {
            console.log('Converting a basket to an order :', response)
        },
    },

    // Add Promotion codes
    {
        break: 'wrong code',
        name: 'Add Promotion codes',
        values: computed(() => {
            return {
                ...url(`/baskets/${basketId.value}/promotion-codes/aWrongPromotionCodeToTest`),
                method: 'POST',
                data: {
                    priceGroupId: priceGroupId.value,
                    cultureCode,
                },
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
    },
    {
        break: 'priceGroupId',
        name: 'Add Promotion codes',
        values: computed(() => {
            return {
                ...url(`/baskets/${basketId.value}/promotion-codes/summer`),
                method: 'POST',
                data: {
                    priceGroupId: '878b0ff1-0800-40ac-b006-0e9858f234a5',
                    cultureCode,
                },
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
    },
    {
        name: 'Add Promotion codes',
        values: computed(() => {
            return {
                ...url(`/baskets/${basketId.value}/promotion-codes/summer`),
                method: 'POST',
                data: {
                    priceGroupId: priceGroupId.value,
                    cultureCode,
                },
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
        success: (response) => {
            console.log('Adding Promotion codes :', response)
        },
    },
    {
        name: 'Promotion codes',
        values: computed(() => {
            return {
                ...url(`/baskets/${basketId.value}/promotion-codes`),
                method: 'GET',
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
        success: (response) => {
            console.log('Getting Promotion codes :', response)
        },
    },
    {
        name: 'Deleting Promotion codes',
        values: computed(() => {
            return {
                ...url(`/baskets/${basketId.value}/promotion-codes/summer`),
                method: 'DELETE',
                headers: {
                    Authorization: accessToken.value,
                },
                data: {
                    priceGroupId: priceGroupId.value,
                    cultureCode,
                },
            }
        }),
        success: (response) => {
            console.log('Getting Promotion codes :', response)
        },
    },

    // T Create Basket Property
    {
        break: 'PriceGroupId',
        name: 'Create Basket Property',
        values: computed(() => {
            return {
                ...url(`/baskets/${basketId.value}/properties`),
                method: 'POST',
                data: {
                    PriceGroupId: '878b0ff1-0800-40ac-b006-0e9858f234a5',
                    cultureCode,
                    key: 'property key',
                    value: 'property value',
                },
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
    },
    {
        name: 'Create Basket Property',
        values: computed(() => {
            return {
                ...url(`/baskets/${basketId.value}/properties`),
                method: 'POST',
                data: {
                    PriceGroupId: priceGroupId.value,
                    cultureCode,
                    key: 'property key',
                    value: 'property value',
                },
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
        success: (response) => {
            console.log('Create Basket Property :', response)
        },
    },
    {
        name: 'Get Basket Properties',
        values: computed(() => {
            return {
                ...url(`/baskets/${basketId.value}/properties`),
                method: 'GET',
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
        success: (response) => {
            basketCustomPropertyId.value = response.data.customProperties[0].id
            console.log('Get Basket Properties :', response)
        },
    },
    {
        break: 'priceGroupId',
        name: 'Update Basket Properties',
        values: computed(() => {
            return {
                ...url(`/baskets/${basketId.value}/properties/${basketCustomPropertyId.value}`),
                method: 'PATCH',
                headers: {
                    Authorization: accessToken.value,
                },
                data: {
                    cultureCode,
                    priceGroupId: fakeGuid,
                    value: 'new custom value',
                },
            }
        }),
    },
    {
        name: 'Update Basket Properties',
        values: computed(() => {
            return {
                ...url(`/baskets/${basketId.value}/properties/${basketCustomPropertyId.value}`),
                method: 'PATCH',
                headers: {
                    Authorization: accessToken.value,
                },
                data: {
                    cultureCode,
                    priceGroupId: priceGroupId.value,
                    value: 'new custom value',
                },
            }
        }),
        success: (response) => {
            console.log('Get Basket Properties :', response)
        },
    },
    {
        name: 'Delete Basket Properties',
        values: computed(() => {
            return {
                ...url(`/baskets/${basketId.value}/properties/${basketCustomPropertyId.value}`),
                method: 'DELETE',
                headers: {
                    Authorization: accessToken.value,
                },
                data: {
                    PriceGroupId: priceGroupId.value,
                    cultureCode,
                }
            }
        }),
        success: (response) => {
            console.log('Get Basket Properties :', response)
        },
    },
    {
        name: 'Available views',
        values: computed(() => {
            return {
                ...url(`/baskets/${basketId.value}/lines?views=miniBasket`),
                method: 'POST',
                data: {
                    // Quantity: 1,
                    Sku: '200-000-001',
                    VariantSku: '002',
                    PriceGroupId: priceGroupId.value, //'878b0ff1-0800-40ac-b006-0e9858f234a5'
                    ProductCatalogId: catalogId.value,
                    cultureCode,
                },
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
        success: (response) => {
            lineId.value = response.data.miniBasket.orderLines[0].id
            console.log('Available views :', response, 'Line Id: '+lineId.value )
        },
    },
    {
        name: 'Add custom properties to orderLine',
        values: computed(() => {
            return {
                ...url(`/baskets/${basketId.value}/lines/${lineId.value}/properties`),
                method: 'POST',
                data: {
                    cultureCode,
                    PriceGroupId: priceGroupId.value,
                    key: 'lineCustomProperty key',
                    value: 'lineCustomProperty value',
                },
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
        success: (response) => {
            console.log('Add custom properties to orderLine :', response)
        },
    },
    {
        name: 'Get custom properties of orderLine',
        values: computed(() => {
            return {
                ...url(`/baskets/${basketId.value}/lines/${lineId.value}/properties`),
                method: 'GET',
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
        success: (response) => {
            orderLineCustomPropertyId.value=response.data.customProperties[0].id
            console.log('Get custom properties of orderLine:', response)
        },
    },
    {
        break: 'priceGroupId',
        name: 'Update custom properties of orderLine',
        values: computed(() => {
            return {
                ...url(`/baskets/${basketId.value}/lines/${lineId.value}/properties/${orderLineCustomPropertyId.value}`),
                method: 'PATCH',
                headers: {
                    Authorization: accessToken.value,
                },
                data: {
                    cultureCode,
                    priceGroupId: fakeGuid,
                    value: 'new value of custom',
                },
            }
        }),
    },
    {
        name: 'Update custom properties of orderLine',
        values: computed(() => {
            return {
                ...url(`/baskets/${basketId.value}/lines/${lineId.value}/properties/${orderLineCustomPropertyId.value}`),
                method: 'PATCH',
                headers: {
                    Authorization: accessToken.value,
                },
                data: {
                    cultureCode,
                    priceGroupId: priceGroupId.value,
                    value: 'new value of custom',
                },
            }
        }),
        success: (response) => {
            console.log('Update custom properties of orderLine:', response)
        },
    },
    {
        name: 'Delete custom properties of orderLine',
        values: computed(() => {
            return {
                ...url(`/baskets/${basketId.value}/lines/${lineId.value}/properties/${orderLineCustomPropertyId.value}`),
                method: 'DELETE',
                headers: {
                    Authorization: accessToken.value,
                },
                data: {
                    cultureCode,
                    priceGroupId: priceGroupId.value,
                },
            }
        }),
        success: (response) => {
            console.log('Delete custom properties of orderLine:', response)
        },
    },
    {
        name: 'Get UPDATED custom properties of orderLine',
        values: computed(() => {
            return {
                ...url(`/baskets/${basketId.value}/lines/${lineId.value}/properties`),
                method: 'GET',
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
        error: true,
        success: (response) => {
            console.log('Get UPDATED custom properties of orderLine:', response)
        },
    },
    {
        name: 'Updating a line item on a basket',
        values: computed(() => {
            return {
                ...url(`/baskets/${basketId.value}/lines/${lineId.value}`),
                method: 'PATCH',
                data: {
                    price: 50,
                },
                headers: {
                    Authorization: accessToken.value,
                },
                params: {
                    PriceGroupId: priceGroupId.value,
                    cultureCode,
                }
            }
        }),
        success: (response) => {
            console.log('Updating a line item on a basket :', response)
        },
    },
    {
        name: 'Deleting a line item on a basket',
        values: computed(() => {
            return {
                ...url(`/baskets/${basketId.value}/lines/${lineId.value}`),
                method: 'DELETE',
                headers: {
                    Authorization: accessToken.value,
                },
                data: {
                    PriceGroupId: priceGroupId.value,
                    cultureCode,
                }
            }
        }),
        success: (response) => {
            console.log('Deleting a line item on a basket :', response)
        },
    },
    {
        name: 'Available views',
        values: computed(() => {
            return {
                ...url(`/baskets/${basketId.value}/lines?views=miniBasket`),
                method: 'POST',
                data: {
                    // Quantity: 1,
                    Sku: '200-000-001',
                    VariantSku: '002',
                    PriceGroupId: priceGroupId.value, //'878b0ff1-0800-40ac-b006-0e9858f234a5'
                    ProductCatalogId: catalogId.value,
                    cultureCode,
                },
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
        success: (response) => {
            console.log('Available views :', response, 'Line Id: '+lineId.value )
        },
    },
    {
        name: 'Get order',
        loading: false,
        error: {message: 'Need to set an order ID'},
        // Geeting order needs fix to get order Id 
        // 
        values: computed(() => {
            return {
                // ...url(`/orders/{orderID}/`),
                method: 'GET',
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
        success: (response) => {
            console.log('Get order:', response)
        },
    },
])

const appStarted = ref(false)

async function sendRequests () {
    appStarted.value = true
    for (const request of sendingRequestArray.value) {
        if (!request?.values?.url) continue

        await axios.request({
            url: request.values.url,
            method: request.values.method,
            data: request.values.data,
            headers: request.values.headers,
            params: request.values.params,
            auth: request.values.auth,
        }).then(response => {
            request.loading = false
            request.result = response
            if (request.success) {
                request.success(response)
            }
        }).catch(error => {
            console.error(request.name ,error)
            request.loading = false
            request.error = error
        })
    }
}

</script>

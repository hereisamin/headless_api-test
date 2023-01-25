<template>
      <CheckList v-for="request in requestArray" :key="request.name"
        :loading="request.loading"
        :error="request.error"
        :request-name="request.name"
        :result="request.result" />
</template>

<script setup>
import CheckList from '../CheckList.vue'
import axios from 'axios'
import { computed, onMounted, reactive, ref} from 'vue'

const props = defineProps({
    token: {
        type: Object,
        default: () => {},
    }
})

const url = function (extend) { return {url: `https://localhost:7102/api/v1${extend}`} }
const basketId = ref('')
const cultureCode = 'en-US'
const currency = 'DKK'
const newToken = ref({})
const accessToken = computed(() => newToken.value.access_token ? 'Bearer ' + newToken.value.access_token : 'Bearer ' + props.token.access)
const countryId = ref('')
const paymentMethodId = ref('')

const requestArray = ref([
    {
        name: 'Post Refresh Token',
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
            console.log('Post Refresh Token Success:', response)
        },
    },
    {
        name: 'Post Create Basket',
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
            basketId.value = response.data.basketId
            console.log('Post Create Basket Success: ', response)
        },
    },
    {
        name: 'Get Basket',
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
            console.log('Get Basket Success: ', response)
        },
    },
    {
        name: 'Get Peice Groups',
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
            console.log('Get Price Group Success: ', response)
        },
    },
    {
        name: 'Get Countries',
        values: computed(() => {
            return {
                ...url('/countries'),
                headers: {
                    Authorization: accessToken.value,
                },      
            }
        }),
        success: (response) => {
            countryId.value = response.data.countries[0].id
            console.log('Get Countries Success: ', response)
        },
    },
    {
        name: 'Get Payment Methods',
        values: computed(() => {
            return {
                ...url('/payment-methods'),
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
            console.log('Get Payment Methods Success: ', response)
        },
    },
    {
        name: 'Get Catalogs',
        values: computed(() => {
            return {
                ...url('/catalogs'),
                headers: {
                    Authorization: accessToken.value,
                },
                params:{
                    cultureCode,
                    priceGroupId: '1127af1d-32a5-6dda-bde4-500a351ff478',
                },
            }
        }),
        success: (response) => {
            console.log('Get Catalogs Success: ', response)
        },
    },
    {
        name: 'Get shipping Methods ??',
    },
    {
        name: 'Post Adding a line item to a basket',
        values: computed(() => {
            return {
                ...url(`/baskets/${ basketId.value }/lines`),
                method: 'POST',
                data: {
                    Quantity: -20,
                    Sku: 'BWWMSFSS-LE',
                    VariantSku: 'BWWMSFSS-LE-15-White',
                    PriceGroupId: '878b0ff1-0800-40ac-b006-0e9858f234a5',
                    ProductCatalogId: '7d27a01f-21e1-4adb-afc2-490c284bd478',
                    cultureCode,
                },
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
        success: (response) => {
            console.log('Post Adding a line item to a basket Success: ', response)
        },
    },
    {
        name: 'Post Add shipping Information',
        values: computed(() => {
            return {
                ...url(`/baskets/${ basketId.value }/shipping`),
                method: 'POST',
                data: {
                    CultureCode: 'en-GB',
                    priceGroupId: '878b0ff1-0800-40ac-b006-0e9858f234a5',
                    shippingMethodId: '3e77be5c-bcee-4688-94a8-a02987f218f3',
                    shippingAddress: {
                        city: 'Aarhus',
                        countryId: '9efae45f-530b-4396-b87c-c34b88169897',
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
            console.log('Post Add shipping Information Success: ', response)
        },
    },
    {
        name: 'Post Add billing address',
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
            console.log('Post Add billing address Success: ', response)
        },
    },
    {
        name: 'Post Converting a basket to an order',
        values: computed(() => {
            return {
                ...url(`/payments`),
                method: 'POST',
                data: {
                    basketId: basketId.value,
                    paymentMethodId: paymentMethodId.value,
                    priceGroupId: '878b0ff1-0800-40ac-b006-0e9858f234a5',
                    cultureCode,
                },
                headers: {
                    Authorization: accessToken.value,
                },
            }
        }),
        success: (response) => {
            console.log('Post Add billing address Success: ', response)
        },
    },
    
])

onMounted(async () => {
    for (const request of requestArray.value) {
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
            request?.success(response)
        }).catch(error => {
            console.error(error)
        request.loading = false
        request.error = error.response.data.errors ?? error
    })
    }
})

</script>

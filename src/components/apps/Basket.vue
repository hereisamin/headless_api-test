<template>
      <CheckList v-for="request in requestArray" :key="request.name"
        :loading="request.loading"
        :error="request.error"
        :request-name="request.name"
        :method="request.values.method"
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
const currency = 'EUR'
const newToken = ref({})
const accessToken = computed(() => newToken.value.access_token ? 'Bearer ' + newToken.value.access_token : 'Bearer ' + props.token.access)
const countryId = ref('')
const paymentMethodId = ref('')
const priceGroupId = ref('')
const catalogId = ref('')
const shippingMethodId = ref('')
const lineId = ref('')

const requestArray = ref([
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
            basketId.value = response.data.basketId
            console.log('Create Basket :', response)
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
        name: 'Peice Groups',
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
                    priceGroupId: priceGroupId.value, //'1127af1d-32a5-6dda-bde4-500a351ff854'
                },
            }
        }),
        success: (response) => {
            catalogId.value = response.data.catalogs[0].id
            console.log('Catalogs :', response)
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
        name: 'Adding a line item to a basket',
        values: computed(() => {
            return {
                ...url(`/baskets/${ basketId.value }/lines`),
                method: 'POST',
                data: {
                    Quantity: 1,
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
            console.log('Adding a line item to a basket :', response)
        },
    },
    {
        name: 'Add shipping Information',
        values: computed(() => {
            return {
                ...url(`/baskets/${ basketId.value }/shipping`),
                method: 'POST',
                data: {
                    cultureCode,
                    priceGroupId: priceGroupId.value, //'878b0ff1-0800-40ac-b006-0e9858f234a5'
                    shippingMethodId: shippingMethodId.value, // '3e77be5c-bcee-4688-94a8-a02987f218f3'
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
            console.log('Add shipping Information :', response)
        },
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
    {
        name: 'Converting a basket to an order',
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
            console.log('Converting a basket to an order :', response)
        },
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
    {
        name: 'Available views',
        values: computed(() => {
            return {
                ...url(`/baskets/${basketId.value}/lines?views=miniBasket`),
                method: 'POST',
                data: {
                    Quantity: 1,
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
        name: 'Updating a line item on a basket',
        values: computed(() => {
            return {
                ...url(`/baskets/${basketId.value}/lines/${lineId.value}`),
                method: 'PATCH',
                data: {
                    quantity: 5,
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
            lineId.value = response.data.miniBasket.orderLines[1].id
            console.log('Available views :', response, 'Line Id: '+lineId.value )
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
                params: {
                    PriceGroupId: priceGroupId.value,
                    cultureCode,
                }
            }
        }),
        success: (response) => {
            lineId.value = response.data.miniBasket.orderLines[1].id
            console.log('Available views :', response, 'Line Id: '+lineId.value )
        },
    },
])

// curl -D- -X DELETE https://umbracodemo1.ucommerce.net/api/v1/baskets/{baketId}/lines/{lineId} \
//     -H 'Authorization: Bearer <ACCESS_TOKEN>'
//     -H 'Content-Type: application/json' \
//     -d '{
//             "PriceGroupId": "878b0ff1-0800-40ac-b006-0e9858f234a5",
//             "CultureCode": "en-US"
//         }'

// curl -D- -X PATCH https://umbracodemo1.ucommerce.net/api/v1/baskets/{baketId}/lines/{lineId}?PriceGroupId={id}&CultureCode={culture} \
//     -H 'Authorization: Bearer <ACCESS_TOKEN>'
//     -H 'Content-Type: application/json' \
//     -d '{
//             "Quantity": {quantity}
//             "Price": {price},
//             "VatRate": "{vatRate}"           
//         }'
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
            console.error(request.name ,error)
        request.loading = false
        request.error = error.response.data.errors ?? error
    })
    }
})

</script>

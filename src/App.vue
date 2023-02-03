<template>
  <div class="w-full  xl:container xl:max-w-6xl min-h-screen shadow-md">
    <TheHeader v-bind="{baseUrl}" />

    <CheckList v-for="request in requests" :key="request.name"
    :request="request"
    :loading="request.loading"
    :error="request.error"
    :request-name="request.name"
    :method="request.method"
    :result="request.result" />

    <Basket v-if="token.access" v-bind="{token}"/>
  </div>
</template>

<script setup>
import {computed, onMounted, reactive, ref} from 'vue'
import axios from 'axios'
import TheHeader from './components/TheHeader.vue'
import CheckList from './components/CheckList.vue'
import Basket from './components/apps/Basket.vue'

const clientUrl = 'http://192.168.1.117:8080' //'http://127.0.0.1:5173'
const baseUrl = 'http://192.168.1.117:5079'

const clientSecret = '99F0E112-586E-44B4-A22B-F5C687E76889pkPdRblQH1i389c7wJG4gsLcpTDKouTLPZ6630T8hXDa7uTBvuxUYC1QEQn6cLGfdNCKTEj9Gu7SSe2XKRFsvL9Es6INsCO7OTUCuI8c55uKhtNz58KhFzG0DpjW2C2BkN'
const client_id = 'c9baba06-d0d7-4f60-812a-f9d16b3f098c' // 'c9baba06-d0d7-4f60-812a-f9d16b3f068d' //
const redirect_uri = clientUrl
const connectEndPoint = '/api/v1/oauth/connect'
const response_type = 'code'
const grant_type = 'authorization_code'
const tokenEndPoint = '/api/v1/oauth/token'

const getCodeUrl = computed(() => `${baseUrl}${connectEndPoint}?client_id=${client_id}&redirect_uri=${redirect_uri}&response_type=${response_type}&scope=transactions:custom:price`)
const getTokenUrl = computed(() => baseUrl+tokenEndPoint)
const credentials = computed(()=> 'Basic '+btoa(`${client_id}:${clientSecret}`))

const requests = reactive({})
const token = reactive({
  client: {
    id: client_id,
    secret: clientSecret,
  },
})

onMounted(() => {
  getTokenAxiosAsync()
})

async function getTokenAxiosAsync() {
  const code = await getCodeAsync()

    requests.getToken = {
    name: 'Post Token Axios',
    loading: true,
    error: false,
    result: null,
    method: 'POST',
    values: {
      url: baseUrl+tokenEndPoint,
      method: 'POST',
      redirect: 'manual',
      headers: {
        Authorization: credentials.value,
        'content-type': 'application/x-www-form-urlencoded',
      },
      data: {
        'code': code,
        'grant_type': grant_type,
        'redirect_uri': redirect_uri,
      }
    }

  }
    await axios.post(getTokenUrl.value, {
      'code': code,
      'grant_type': grant_type,
      'redirect_uri': redirect_uri,
    },{
    headers: {
      Authorization: credentials.value,
      'content-type': 'application/x-www-form-urlencoded',
    },
    redirect: 'manual',
  }).then(response => {
      token.access = response.data['access_token']
      token.expires = response.data['expires_in']
      token.refresh = response.data['refresh_token']
      token.type = response.data['token_type']
      requests.getToken.result = response
      requests.getToken.loading = false
  }).catch(error => {
      requests.getToken.loading = false
      requests.getToken.error = error
  })
}

async function getCodeAsync() {
  requests.getCode = {
    name: 'Get Code',
    loading: true,
    error: null,
    result: null,
    method: 'GET',
    values: {
      url: `${baseUrl}${connectEndPoint}`,
      method: 'GET',
      redirect: 'follow',
      params: {
        client_id: client_id,
        redirect_uri: redirect_uri,
        response_type: response_type,
        scope: 'transactions:custom:price',
      }
    }
  }
  let code = ''

  const response = await fetch(getCodeUrl.value, {
    method: 'GET',
    redirect: 'follow',
  }).then((response) => {
    const urlSearchParams = new URLSearchParams(response.url)
    code = urlSearchParams.get(clientUrl+'/?code')
    requests.getCode.result = {url: response.url}
    requests.getCode.loading = false
  }).catch(error => {
    requests.getCode.error = error
    requests.getToken.loading = false
  })
  return code
}
</script>

<style>

  input:checked {
    background-color: #22c55e; /* bg-green-500 */
  }

  input:checked ~ span:last-child {
    --tw-translate-x: 1.75rem; /* translate-x-7 */
  }
</style>
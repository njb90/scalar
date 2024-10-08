<script setup lang="ts">
import { DataTableRow } from '@/components/DataTable'
import { type UpdateScheme, useWorkspace } from '@/store/workspace'
import RequestAuthDataTableInput from '@/views/Request/RequestSection/RequestAuthDataTableInput.vue'
import { authorizeOauth2 } from '@/views/Request/libs'
import { ScalarButton, useLoadingState } from '@scalar/components'
import type { SecuritySchemeOauth2 } from '@scalar/oas-utils/entities/workspace/security'

import OAuthScopesInput from './OAuthScopesInput.vue'

const props = defineProps<{
  scheme: SecuritySchemeOauth2
}>()

const loadingState = useLoadingState()
const { isReadOnly, securitySchemeMutators } = useWorkspace()

/** Update the current scheme */
const updateScheme: UpdateScheme = (path, value) =>
  securitySchemeMutators.edit(props.scheme.uid, path, value)

/** Authorize the user using specified flow */
const handleAuthorize = async () => {
  if (loadingState.isLoading) return
  loadingState.startLoading()

  const accessToken = await authorizeOauth2(props.scheme).finally(() =>
    loadingState.stopLoading(),
  )

  if (accessToken) updateScheme('flow.token', accessToken)
}
</script>

<template>
  <!-- Access Token Granted -->
  <template v-if="scheme.flow.token">
    <DataTableRow>
      <RequestAuthDataTableInput
        id="oauth2-access-token"
        class="border-r-transparent"
        :modelValue="scheme.flow.token"
        placeholder="QUxMIFlPVVIgQkFTRSBBUkUgQkVMT05HIFRPIFVT"
        type="password"
        @update:modelValue="(v) => updateScheme('flow.token', v)">
        Access Token
      </RequestAuthDataTableInput>
    </DataTableRow>
    <DataTableRow class="min-w-full">
      <div class="h-8 flex items-center justify-self-end">
        <ScalarButton
          class="p-0 py-0.5 px-2 mr-1"
          :loading="loadingState"
          size="sm"
          variant="outlined"
          @click="updateScheme('flow.token', '')">
          Clear
        </ScalarButton>
      </div>
    </DataTableRow>
  </template>

  <template v-else>
    <!-- Custom auth -->
    <DataTableRow v-if="!isReadOnly">
      <RequestAuthDataTableInput
        v-if="'authorizationUrl' in scheme.flow"
        :id="`oauth2-authorization-url-${scheme.uid}`"
        :modelValue="scheme.flow.authorizationUrl"
        placeholder="https://galaxy.scalar.com/authorize"
        @update:modelValue="(v) => updateScheme('flow.authorizationUrl', v)">
        Authorization Url
      </RequestAuthDataTableInput>

      <RequestAuthDataTableInput
        v-if="'tokenUrl' in scheme.flow"
        :id="`oauth2-token-url-${scheme.uid}`"
        :modelValue="scheme.flow.tokenUrl"
        placeholder="https://galaxy.scalar.com/token"
        @update:modelValue="(v) => updateScheme('flow.tokenUrl', v)">
        Token Url
      </RequestAuthDataTableInput>
    </DataTableRow>

    <DataTableRow v-if="'redirectUri' in scheme.flow">
      <!-- Redirect URI -->
      <RequestAuthDataTableInput
        :id="`oauth2-redirect-uri-${scheme.uid}`"
        :modelValue="scheme.flow.redirectUri"
        placeholder="https://galaxy.scalar.com/callback"
        @update:modelValue="(v) => updateScheme('flow.redirectUri', v)">
        Redirect Url
      </RequestAuthDataTableInput>
    </DataTableRow>

    <!-- Username and password -->
    <template v-if="scheme.flow.type === 'password'">
      <DataTableRow>
        <RequestAuthDataTableInput
          :id="`oauth2-password-username-${scheme.uid}`"
          class="text-c-2"
          :modelValue="scheme.flow.value"
          placeholder="Key"
          @update:modelValue="(v) => updateScheme('flow.value', v)">
          Key
        </RequestAuthDataTableInput>
      </DataTableRow>
      <DataTableRow>
        <RequestAuthDataTableInput
          :id="`oauth2-password-password-${scheme.uid}`"
          :modelValue="scheme.flow.secondValue"
          placeholder="xxxxxx"
          type="password"
          @update:modelValue="(v) => updateScheme('flow.secondValue', v)">
          Secret
        </RequestAuthDataTableInput>
      </DataTableRow>
    </template>

    <!-- Client ID -->
    <DataTableRow>
      <RequestAuthDataTableInput
        :id="`oauth2-client-id-${scheme.uid}`"
        :modelValue="scheme.clientId"
        placeholder="12345"
        @update:modelValue="(v) => updateScheme('clientId', v)">
        Client ID
      </RequestAuthDataTableInput>
    </DataTableRow>

    <!-- Client Secret (Authorization Code / Client Credentials / Password (optional)) -->
    <DataTableRow v-if="'clientSecret' in scheme.flow">
      <RequestAuthDataTableInput
        :id="`oauth2-client-secret-${scheme.uid}`"
        :modelValue="scheme.flow.clientSecret"
        placeholder="XYZ123"
        type="password"
        @update:modelValue="(v) => updateScheme('flow.clientSecret', v)">
        Client Secret
      </RequestAuthDataTableInput>
    </DataTableRow>

    <!-- Scopes -->
    <DataTableRow v-if="scheme.flow.scopes">
      <OAuthScopesInput
        :activeFlow="scheme.flow"
        :updateScheme="updateScheme" />
    </DataTableRow>

    <DataTableRow class="min-w-full">
      <div class="h-8 flex items-center justify-self-end">
        <ScalarButton
          class="p-0 py-0.5 px-2 mr-1"
          :loading="loadingState"
          size="sm"
          variant="outlined"
          @click="handleAuthorize">
          Authorize
        </ScalarButton>
      </div>
    </DataTableRow>

    <!-- Open ID Connect -->
    <!-- <DataTableRow -->
    <!--   v-else-if="activeScheme?.type === 'openIdConnect'" -->
    <!--   class="border-r-transparent"> -->
    <!--   <DataTableInput -->
    <!--     v-model="password" -->
    <!--     placeholder="Token"> -->
    <!--     TODO -->
    <!--   </DataTableInput> -->
    <!--   <DataTableCell class="flex items-center"> -->
    <!--     <ScalarButton size="sm"> Authorize </ScalarButton> -->
    <!--   </DataTableCell> -->
    <!-- </DataTableRow> -->
  </template>
</template>

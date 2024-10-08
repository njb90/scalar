<script lang="ts" setup>
import { ScalarIcon } from '@scalar/components'
import type { OpenAPIV2, OpenAPIV3, OpenAPIV3_1 } from '@scalar/openapi-types'
import { computed, onMounted, onServerPrefetch } from 'vue'

import { useAuthenticationStore } from '../stores'

const props = defineProps<{
  value?:
    | OpenAPIV2.SecurityDefinitionsObject
    | OpenAPIV3.ComponentsObject['securitySchemes']
    | OpenAPIV3_1.ComponentsObject['securitySchemes']
}>()

// Emit updates
const emits = defineEmits<{
  (event: 'input', key: string): void
}>()

const { authentication, setAuthentication } = useAuthenticationStore()

// Update credentials in state
const handleAuthenticationTypeInput = (event: Event) => {
  setSecuritySchemeKey((event.target as HTMLSelectElement).value)
}

// Use first security scheme as default
onMounted(() => {
  // Oh, the key was set already!
  if (authentication.preferredSecurityScheme) {
    return
  }

  // Set the authentication type to the first security scheme
  setSecuritySchemeKey(Object.keys(props.value ?? {})[0] ?? null)
})

// Update current security scheme key
const setSecuritySchemeKey = (key: string) => {
  setAuthentication({
    preferredSecurityScheme: key,
  })

  emits('input', key)
}

onServerPrefetch(() =>
  setSecuritySchemeKey(Object.keys(props.value ?? {})[0] ?? null),
)

const isNone = (item: any) => !item?.type

const isApiKey = (item: any) => item.type.toLowerCase() === 'apikey'

const isHttpBasic = (item: any) =>
  (item.type === 'http' && item.scheme.toLowerCase() === 'basic') ||
  item.type.toLowerCase() === 'basic'

const isHttpBearer = (item: any) =>
  item.type === 'http' && item.scheme.toLowerCase() === 'bearer'

const isOAuth2 = (item: any) => item.type.toLowerCase() === 'oauth2'

// Translate type to label
const getLabelForScheme = (item: any, key: string) => {
  return `${getAuthorizationTypeLabel(item)} (${key})`
}

// Translate type to label
const getAuthorizationTypeLabel = (item: any) => {
  if (isNone(item)) {
    return 'No Authentication'
  } else if (isApiKey(item)) {
    return 'API Key'
  } else if (isHttpBasic(item)) {
    return 'Basic Authentication'
  } else if (isHttpBearer(item)) {
    return 'Bearer Authentication'
  } else if (isOAuth2(item)) {
    if (item.flows?.implicit) {
      return 'OAuth 2.0 Implicit'
    }

    if (item.flows?.password) {
      return 'OAuth 2.0 Password'
    }

    return 'OAuth 2.0'
  }

  return `${item.type} (not yet supported)`
}

// Alias
const keys = computed(() => Object.keys(props.value ?? {}))
</script>
<template>
  <!-- Single security scheme -->
  <template v-if="keys.length === 1">
    <!-- Use <div> to avoid unnecessary styles added by `CollapsibleSection` -->
    <div class="security-scheme-label">
      {{ getLabelForScheme(value?.[keys[0]], keys[0]) }}
    </div>
  </template>

  <!-- Multiple security schemes -->
  <template v-else-if="keys.length > 1">
    <div class="security-scheme-selector">
      <span class="security-scheme-label">
        {{
          authentication.preferredSecurityScheme
            ? getLabelForScheme(
                value?.[authentication.preferredSecurityScheme],
                authentication.preferredSecurityScheme,
              )
            : 'No Authentication'
        }}
      </span>
      <ScalarIcon icon="ChevronDown" />
      <select
        :value="authentication.preferredSecurityScheme"
        @click.prevent
        @input="handleAuthenticationTypeInput">
        <option value="">No Authentication</option>
        <template
          v-for="key in keys"
          :key="key">
          <option :value="key ?? null">
            {{ getLabelForScheme(value?.[key], key) }}
          </option>
        </template>
      </select>
    </div>
  </template>
</template>

<style scoped>
.security-scheme-selector {
  position: relative;
  display: flex;
  border-radius: var(--scalar-radius);
  color: var(--scalar-color-3);
  display: flex;
  align-items: center;
  gap: 4px;
  cursor: pointer;
}
.security-scheme-selector:hover {
  color: var(--scalar-color-1);
}
.security-scheme-label {
  color: var(--scalar-color-3);
  font-size: var(--scalar-mini);
}
.security-scheme-selector:hover .security-scheme-label {
  color: var(--scalar-color-1);
}
.security-scheme-selector select {
  position: absolute;
  cursor: pointer;
  opacity: 0;
  right: 0;
  -moz-appearance: none;
  -webkit-appearance: none;
  appearance: none;
  width: 100%;
  /** Increase clickable area */
  margin-top: -5px;
  padding: 10px 0;
}

.security-scheme-selector svg {
  width: 12px;
  stroke: currentColor;
}
</style>

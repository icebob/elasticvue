<template>
  <v-list-item>
    <v-list-item-content>{{ text }}</v-list-item-content>
    <v-list-item-action>
      <v-btn :id="name" :color="color" :loading="loading" @click.native="confirmMethod">{{
          $t('utilities.run')
        }}
      </v-btn>
    </v-list-item-action>
  </v-list-item>
</template>

<script>
  import { showDefaultSnackbar, showSnackbar } from '@/mixins/ShowSnackbar'
  import { computed, ref } from '@vue/composition-api'
  import { useElasticsearchRequest } from '@/mixins/RequestComposition'
  import i18n from '@/i18n'

  export default {
    class: 'utility',
    props: {
      text: {
        default: '',
        type: String
      },
      method: {
        default: '',
        type: String
      },
      methodParams: {
        default: () => {
        },
        type: [Object, Array]
      },
      confirmMessage: {
        default: '',
        type: String
      },
      name: {
        default: 'utility',
        type: String
      }
    },
    setup (props) {
      const loading = ref(false)
      const { requestState, callElasticsearch } = useElasticsearchRequest()

      const runMethod = () => {
        callElasticsearch(props.method, props.methodParams)
          .then(body => {
            if (body) {
              showSnackbar(requestState.value, { title: i18n.t('defaults.success'), body: JSON.stringify(body) })
            } else {
              showDefaultSnackbar({ title: i18n.t('defaults.success') })
            }
          })
      }

      const color = computed(() => {
        return requestState.apiError || requestState.networkError ? 'red' : 'primary-button'
      })

      const confirmMethod = () => {
        if (props.confirmMessage) {
          if (confirm(props.confirmMessage)) {
            runMethod()
          }
        } else {
          runMethod()
        }
      }

      return {
        loading,
        color,
        confirmMethod
      }
    }
  }
</script>

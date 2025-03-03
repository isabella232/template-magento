<template>
  <ValidationObserver v-slot="{ handleSubmit, reset }">
    <form
      class="form"
      @submit.prevent="handleSubmit(submitForm(reset))"
    >
      <ValidationProvider
        v-slot="{ errors }"
        rules="required"
        class="form__element"
      >
        <SfInput
          v-model="form.currentPassword"
          type="password"
          name="currentPassword"
          :label="$t('Current Password')"
          required
          :valid="!errors[0]"
          :error-message="errors[0]"
        />
      </ValidationProvider>
      <div class="form__horizontal">
        <ValidationProvider
          v-slot="{ errors }"
          rules="required|password"
          vid="password"
          class="form__element"
        >
          <SfInput
            v-model="form.newPassword"
            type="password"
            name="newPassword"
            :label="$t('New Password')"
            required
            :valid="!errors[0]"
            :error-message="errors[0]"
          />
        </ValidationProvider>
        <ValidationProvider
          v-slot="{ errors }"
          rules="required|confirmed:password"
          class="form__element"
        >
          <SfInput
            v-model="form.repeatPassword"
            type="password"
            name="repeatPassword"
            :label="$t('Repeat Password')"
            required
            :valid="!errors[0]"
            :error-message="errors[0]"
          />
        </ValidationProvider>
      </div>
      <SfButton class="form__button">
        {{ $t('Update password') }}
      </SfButton>
    </form>
  </ValidationObserver>
</template>

<script>
import { defineComponent, ref } from '@nuxtjs/composition-api';
import { ValidationProvider, ValidationObserver } from 'vee-validate';
import { SfInput, SfButton } from '@storefront-ui/vue';
import { useUiNotification } from '~/composables';

export default defineComponent({
  name: 'PasswordResetForm',

  components: {
    SfInput,
    SfButton,
    ValidationProvider,
    ValidationObserver,
  },

  setup(_, { emit }) {
    const resetForm = () => ({
      currentPassword: '',
      newPassword: '',
      repeatPassword: '',
    });
    const {
      send: sendNotification,
    } = useUiNotification();

    const form = ref(resetForm());

    const submitForm = (resetValidationFn) => () => {
      const onComplete = () => {
        form.value = resetForm();
        sendNotification({
          id: Symbol('password_updated'),
          message: 'The user password was changed successfully updated!',
          type: 'success',
          icon: 'check',
          persist: false,
          title: 'User Account',
        });
        resetValidationFn();
      };

      const onError = (msg) => {
        sendNotification({
          id: Symbol('password_not_updated'),
          message: msg,
          type: 'danger',
          icon: 'cross',
          persist: false,
          title: 'User Account',
        });
      };

      emit('submit', { form, onComplete, onError });
    };

    return {
      form,
      submitForm,
    };
  },
});
</script>

<style lang='scss' scoped>
.form {
  &__element {
    display: block;
    margin: 0 0 var(--spacer-lg) 0;
  }
  &__button {
    display: block;
    width: 100%;
    @include for-desktop {
      width: 17.5rem;
    }
  }
  &__horizontal {
    @include for-desktop {
      display: flex;
      flex-direction: row;
      justify-content: space-between;
    }
    .form__element {
      @include for-desktop {
        flex: 1;
        margin-right: var(--spacer-lg);
      }
      &:last-child {
        margin-right: 0;
      }
    }
  }
}
</style>

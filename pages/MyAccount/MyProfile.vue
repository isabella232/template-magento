<template>
  <SfTabs :open-tab="1">
    <!-- Personal data update -->
    <SfTab :title="$t('Personal data')">
      <p class="message">
        {{ $t('Feel free to edit') }}
      </p>

      <ProfileUpdateForm
        :loading="loading"
        @submit="updatePersonalData"
      />

      <p class="notice">
        {{ $t('Use your personal data') }}
        <a href="">{{ $t('Privacy Policy') }}</a>
      </p>
    </SfTab>

    <!-- Password reset -->
    <SfTab :title="$t('Password change')">
      <p class="message">
        {{ $t('Change password your account') }}:<br>
      </p>

      <PasswordResetForm @submit="updatePassword" />
    </SfTab>
  </SfTabs>
</template>

<script>
import { extend } from 'vee-validate';
import {
  required,
  min,
  confirmed,
} from 'vee-validate/dist/rules';
import { SfTabs } from '@storefront-ui/vue';
import { onSSR } from '@vue-storefront/core';
import { useUser } from '@vue-storefront/magento';
import { defineComponent } from '@nuxtjs/composition-api';
import ProfileUpdateForm from '~/components/MyAccount/ProfileUpdateForm.vue';
import PasswordResetForm from '~/components/MyAccount/PasswordResetForm.vue';
import { customerPasswordRegExp, invalidPasswordMsg } from '../../helpers/customer/regex';

extend('required', {
  ...required,
  message: 'This field is required',
});

extend('min', {
  ...min,
  message: 'The field should have at least {length} characters',
});

extend('password', {
  message: invalidPasswordMsg,
  // eslint-disable-next-line @typescript-eslint/no-unsafe-argument
  validate: (value) => customerPasswordRegExp.test(value),
});

extend('confirmed', {
  ...confirmed,
  message: 'Passwords don\'t match',
});

export default defineComponent({
  name: 'PersonalDetails',

  components: {
    SfTabs,
    ProfileUpdateForm,
    PasswordResetForm,
  },

  setup() {
    const {
      changePassword,
      errors,
      load,
      loading,
      updateUser,
      error,
    } = useUser();

    const formHandler = async (fn, onComplete, onError) => {
      await fn();
      const actionErr = error.value.changePassword || error.value.updateUser;
      if (actionErr) {
        onError(actionErr);
      } else {
        onComplete();
      }
    };

    const updatePersonalData = ({
      form,
      onComplete,
      onError,
    }) => formHandler(
      async () => updateUser({ user: form.value }),
      onComplete,
      onError,
    );

    const updatePassword = ({
      form,
      onComplete,
      onError,
    }) => formHandler(async () => changePassword({
      current: form.value.currentPassword,
      new: form.value.newPassword,
    }), onComplete, onError);

    onSSR(async () => {
      await load();
    });

    return {
      loading,
      errors,
      updatePersonalData,
      updatePassword,
    };
  },
});
</script>

<style lang='scss' scoped>
.message,
.notice {
  font-family: var(--font-family--primary);
  line-height: 1.6;
}

.message {
  margin: 0 0 var(--spacer-xl) 0;
  font-size: var(--font-size--base);

  &__label {
    font-weight: 400;
  }
}

.notice {
  margin: var(--spacer-lg) 0 0 0;
  font-size: var(--font-size--sm);
}

</style>

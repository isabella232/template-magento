<template>
  <transition name="fade">
    <SfTabs
      v-if="editingAddress"
      key="edit-address"
      :open-tab="1"
      class="tab-orphan"
    >
      <SfTab
        :title="isNewAddress ? $t('Add the address') : $t('Update the address')"
      >
        <p class="message">
          {{ $t('Contact details updated') }}
        </p>

        <AddressForm
          :address="activeAddress"
          :is-new="isNewAddress"
          @submit="saveAddress"
        />
      </SfTab>
    </SfTabs>

    <SfTabs
      v-else
      key="address-list"
      :open-tab="1"
      class="tab-orphan"
    >
      <SfTab :title="$t('Addresses details')">
        <p class="message">
          {{ $t('Manage addresses') }}
        </p>
        <transition-group
          tag="div"
          name="fade"
          class="addresses-list"
        >
          <div
            v-for="address in userAddresses"
            :key="userAddressesGetters.getId(address)"
            class="addresses"
          >
            <div class="addresses__content">
              <div class="addresses__address">
                <UserAddressDetails :address="address" />
              </div>
            </div>
            <div class="addresses__actions">
              <SfIcon
                icon="cross"
                color="gray"
                size="14px"
                role="button"
                class="smartphone-only"
                @click="removeAddress(address)"
              />
              <SfButton
                @click="changeAddress(address)"
              >
                {{ $t('Change') }}
              </SfButton>

              <SfButton
                v-if="!userAddressesGetters.isDefault(address)"
                class="color-light addresses__button-delete desktop-only"
                @click="removeAddress(address)"
              >
                {{ $t('Delete') }}
              </SfButton>
            </div>
          </div>
        </transition-group>
        <SfButton
          class="action-button"
          @click="changeAddress()"
        >
          {{ $t('Add new address') }}
        </SfButton>
      </SfTab>
    </SfTabs>
  </transition>
</template>
<script>
import {
  SfTabs,
  SfButton,
  SfIcon,
} from '@storefront-ui/vue';
import { userAddressesGetters, useAddresses } from '@vue-storefront/magento';
import {
  computed, defineComponent, useRouter, useRoute, useContext,
} from '@nuxtjs/composition-api';
import { onSSR } from '@vue-storefront/core';
import AddressForm from '~/components/MyAccount/AddressForm.vue';
import UserAddressDetails from '~/components/UserAddressDetails.vue';

export default defineComponent({
  name: 'ShippingDetails',
  components: {
    SfTabs,
    SfButton,
    SfIcon,
    UserAddressDetails,
    AddressForm,
  },
  setup() {
    const {
      addresses,
      load,
      remove,
      update,
      save,
    } = useAddresses();

    const userAddresses = computed(() => userAddressesGetters.getAddresses(addresses.value));
    const router = useRouter();
    const route = useRoute();
    const { app } = useContext();
    const activeAddress = computed(
      () => userAddresses.value.filter((address) => String(address?.id) === route.value.query.id).pop(),
    );

    const getTranslatedUrlAddress = (title) => app.i18n.t(`${title}`).toLowerCase().replace(' ', '-');
    const isNewAddress = computed(() => !activeAddress.value);
    const editingAddress = computed(() => !!route.value.query.id);
    const changeAddress = async (address) => {
      const addressId = address?.id || 'new';
      await router.push(`${app.localePath({ path: `/my-account/${getTranslatedUrlAddress('Addresses details')}`, query: { id: addressId } })}`);
    };

    const removeAddress = async (address) => {
      const isDefault = userAddressesGetters.isDefault(address);
      if (!isDefault) {
        await remove({ address });
        await load({});
      }
    };

    const saveAddress = async ({ form, onComplete, onError }) => {
      try {
        const actionMethod = isNewAddress.value ? save : update;
        const data = await actionMethod({ address: form });
        await onComplete(data);
        await router.push(app.localePath(`/my-account/${getTranslatedUrlAddress('Addresses details')}`));
      } catch (error) {
        onError(error);
      }
    };

    onSSR(async () => {
      await load({});
    });

    return {
      changeAddress,
      update,
      removeAddress,
      saveAddress,
      userAddressesGetters,
      userAddresses,
      editingAddress,
      activeAddress,
      isNewAddress,
    };
  },
});
</script>

<style lang='scss' scoped>

.message {
  font-family: var(--font-family--primary);
  line-height: 1.6;
  font-size: var(--font-size--base);
  margin: 0 0 var(--spacer-base);
}

.addresses-list {
  margin-bottom: var(--spacer-base);
}

.addresses {
  display: flex;
  padding: var(--spacer-xl) 0;
  border-top: 1px solid var(--c-light);

  &:last-child {
    border-bottom: 1px solid var(--c-light);
  }

  &__content {
    flex: 1;
    color: var(--c-text);
    font-size: var(--font-size--base);
    font-weight: 300;
    line-height: 1.6;
  }

  &__actions {
    flex: 1;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    align-items: flex-end;
    @include for-desktop {
      flex-direction: row;
      align-items: center;
      justify-content: flex-end;
    }
  }

  &__button-delete {
    color: var(--c-link);
    @include for-desktop {
      margin-left: var(--spacer-base);
    }
  }

  &__address {
    margin: 0;

    p {
      margin: 0;
    }
  }

  &__client-name {
    font-size: var(--font-size--base);
    font-weight: 500;
  }
}

.action-button {
  width: 100%;
  @include for-desktop {
    width: auto;
  }
}

.tab-orphan {
  @include for-mobile {
    ::v-deep .sf-tabs {
      &__title {
        display: none;
      }

      &__content {
        border: 0;
        padding: 0;
      }
    }
  }
}
</style>

<template>
  <!-- TODO: create logic with isActive prop for BottomNavigationItems -->
  <SfBottomNavigation class="navigation-bottom smartphone-only">
    <SfBottomNavigationItem
      :class="$route.path == '/' ? 'sf-bottom-navigation__item--active' : ''"
      icon="home"
      size="20px"
      label="Home"
      @click="$router.push(app.localePath('/')) && (isMobileMenuOpen ? toggleMobileMenu() : false)"
    />
    <SfBottomNavigationItem
      icon="menu"
      size="20px"
      label="Menu"
      @click="toggleMobileMenu"
    />
    <SfBottomNavigationItem
      v-if="isAuthenticated"
      icon="heart"
      size="20px"
      label="Wishlist"
      @click="toggleWishlistSidebar"
    />
    <SfBottomNavigationItem
      icon="profile"
      size="20px"
      label="Account"
      @click="handleAccountClick"
    />
    <!-- TODO: add logic for label - if on Home then Basket, if on PDC then AddToCart etc. -->
    <SfBottomNavigationItem
      label="Basket"
      icon="add_to_cart"
      @click="toggleCartSidebar"
    >
      <template #icon>
        <SfCircleIcon aria-label="Add to cart">
          <SfIcon
            icon="add_to_cart"
            color="white"
            size="25px"
            :style="{margin: '0 0 0 -2px'}"
          />
        </SfCircleIcon>
      </template>
    </SfBottomNavigationItem>
  </SfBottomNavigation>
</template>

<script>
import { SfBottomNavigation, SfIcon, SfCircleIcon } from '@storefront-ui/vue';
import { useUser } from '@vue-storefront/magento';
import { defineComponent, useRouter, useContext } from '@nuxtjs/composition-api';
import { useUiState } from '~/composables';

export default defineComponent({
  components: {
    SfBottomNavigation,
    SfIcon,
    SfCircleIcon,
  },
  setup() {
    const {
      toggleCartSidebar,
      toggleWishlistSidebar,
      toggleLoginModal,
      toggleMobileMenu,
      isMobileMenuOpen,
    } = useUiState();
    const { isAuthenticated } = useUser();
    const router = useRouter();
    const { app } = useContext();
    const handleAccountClick = async () => {
      if (isAuthenticated.value) {
        await router.push(`${app.localePath('/my-account')}`);
      }
      toggleLoginModal();
    };

    return {
      isAuthenticated,
      isMobileMenuOpen,
      toggleWishlistSidebar,
      toggleCartSidebar,
      toggleMobileMenu,
      handleAccountClick,
      app,
    };
  },
});
</script>
<style lang="scss" scoped>
.navigation-bottom {
  --bottom-navigation-z-index: 3;
}
</style>

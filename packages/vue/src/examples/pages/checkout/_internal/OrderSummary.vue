<template>
  <div>
    <div class="highlighted">
      <SfHeading
        title="Order summary"
        class="sf-heading--left sf-heading--no-underline title"
      />
      <div class="total-items">
        <h3>Total items: {{ totalItems }}</h3>
        <SfButton
          class="sf-button--text color-secondary"
          @click="listIsHidden = !listIsHidden"
          >{{ listIsHidden ? "Show" : "Hide" }} items list
        </SfButton>
      </div>
      <transition name="fade">
        <div v-if="!listIsHidden" class="collected-product-list">
          <SfCollectedProduct
            v-for="(product, index) in products"
            :key="index"
            v-model="product.qty"
            :image="product.image"
            :title="product.title"
            :regular-price="product.price.regular"
            :special-price="product.price.special"
            :stock="product.stock"
            class="collected-product"
            @click:remove="removeProduct(index)"
          >
            <template #configuration>
              <div class="collected-product__properties">
                <SfProperty
                  v-for="(property, key) in product.configuration"
                  :key="key"
                  :name="property.name"
                  :value="property.value"
                  class="collected-product__property"
                />
              </div>
            </template>
            <template #actions>
              <div>
                <div class="collected-product__action">{{ product.sku }}</div>
                <div class="collected-product__action">
                  Quantity: <span class="product__qty">{{ product.qty }}</span>
                </div>
              </div>
            </template>
            <template #input>
              <span></span>
            </template>
          </SfCollectedProduct>
        </div>
      </transition>
    </div>
    <div class="highlighted highlighted--total">
      <SfProperty
        name="Products"
        :value="totalItems"
        class="sf-property--full-width property"
      />
      <SfProperty
        name="Subtotal"
        :value="subtotal"
        class="sf-property--full-width property"
      />
      <SfProperty
        name="Shipping"
        :value="shippingMethod.price"
        class="sf-property--full-width property"
      />
      <SfProperty
        name="Total"
        :value="total"
        class="sf-property--full-width property-total"
      />
    </div>
    <div class="highlighted promo-code">
      <SfButton
        class="sf-button--text promo-code__button "
        @click="showPromoCode = !showPromoCode"
        >{{ showPromoCode ? "-" : "+" }} Promo Code
      </SfButton>
      <transition name="fade">
        <div v-if="showPromoCode">
          <SfInput
            v-model="promoCode"
            name="promoCode"
            label="Enter promo code"
            class="promo-code__input"
          />
          <SfButton class="sf-button--full-width">Apply code</SfButton>
        </div>
      </transition>
    </div>
    <div class="highlighted">
      <SfCharacteristic
        v-for="characteristic in characteristics"
        :key="characteristic.title"
        :title="characteristic.title"
        :description="characteristic.description"
        :icon="characteristic.icon"
        color-icon="green-primary"
        class="characteristic"
      />
    </div>
  </div>
</template>
<script>
import {
  SfHeading,
  SfButton,
  SfCollectedProduct,
  SfProperty,
  SfCharacteristic,
  SfInput
} from "@storefront-ui/vue";

export default {
  name: "OrderSummary",
  components: {
    SfHeading,
    SfButton,
    SfCollectedProduct,
    SfProperty,
    SfCharacteristic,
    SfInput
  },
  props: {
    order: {
      type: Object,
      default: () => ({})
    },
    shippingMethods: {
      type: Array,
      default: () => []
    },
    paymentMethods: {
      type: Array,
      default: () => []
    }
  },
  data() {
    return {
      promoCode: "",
      showPromoCode: false,
      listIsHidden: false,
      characteristics: [
        {
          title: "Safety",
          description: "It carefully packaged with a personal touch",
          icon: "safety"
        },
        {
          title: "Easy shipping",
          description:
            "You’ll receive dispatch confirmation and an arrival date",
          icon: "shipping"
        },
        {
          title: "Changed your mind?",
          description: "Rest assured, we offer free returns within 30 days",
          icon: "return"
        }
      ]
    };
  },
  computed: {
    products() {
      return this.order.products;
    },
    totalItems() {
      return (
        "" +
        this.products.reduce((previous, current) => {
          return previous + current.qty;
        }, 0)
      );
    },
    shipping() {
      return this.order.shipping;
    },
    shippingMethod() {
      const shippingMethod = this.shipping.shippingMethod;
      const method = this.shippingMethods.find(
        method => method.value === shippingMethod
      );
      return method ? method : { price: "$0.00" };
    },
    payment() {
      return this.order.payment;
    },
    paymentMethod() {
      const paymentMethod = this.payment.paymentMethod;
      const method = this.paymentMethods.find(
        method => method.value === paymentMethod
      );
      return method ? method : { label: "" };
    },
    subtotal() {
      const products = this.products;
      const subtotal = products.reduce((previous, current) => {
        const qty = current.qty;
        const price = current.price.special
          ? current.price.special
          : current.price.regular;
        const total = qty * parseFloat(price.replace("$", ""));
        return previous + total;
      }, 0);
      return "$" + subtotal.toFixed(2);
    },
    total() {
      const subtotal = parseFloat(this.subtotal.replace("$", ""));
      const shipping = parseFloat(this.shippingMethod.price.replace("$", ""));
      const total = subtotal + (isNaN(shipping) ? 0 : shipping);
      return "$" + total.toFixed(2);
    }
  },
  methods: {
    removeProduct(index) {
      const order = { ...this.order };
      const products = [...order.products];
      products.splice(index, 1);
      order.products = products;
      this.$emit("update:order", order);
    }
  }
};
</script>
<style lang="scss" scoped>
@import "~@storefront-ui/vue/styles";

.highlighted {
  box-sizing: border-box;
  width: 100%;
  background: #f1f2f3;
  padding: var(--spacer-extra-big);
  margin: 0 0 var(--spacer-big) 0;
  font: 300 var(--font-size-mall) / 1.6 var(--body-font-family-secondary);
  &:last-child {
    margin: 0;
  }
}
.title {
  --heading-title-margin: 0 0 var(--spacer-extra-big) 0;
}
.total-items {
  display: flex;
  align-items: center;
  justify-content: space-between;
  font: 400 var(--font-size-big) / 1.6 var(--body-font-family-secondary);
  h3 {
    font: inherit;
  }
}
.collected-product-list {
  flex: 1;
  margin: var(--spacer-big) calc(var(--spacer-big) * -1);
}
.collected-product {
  margin: var(--spacer-big) 0;
  --collected-product-title-font: 300 var(--font-size-small) / 1.6
    var(--body-font-family-secondary);
  &:last-child {
    margin: var(--spacer-big) 0 0 0;
  }
  &__properties {
    margin: var(--spacer-big) 0 0 0;
  }
  &__property {
    --property-name-font: 300 var(--font-size-extra-small) / 1.6
      var(--body-font-family-secondary);
    --property-value-font: 300 var(--font-size-extra-small) / 1.6
      var(--body-font-family-secondary);
  }
  &__actions {
    transition: opacity 150ms ease-in-out;
    opacity: var(--cp-actions-opacity, 0);
  }
  &__action {
    --button-padding: 0;
    font: 300 var(--font-size-extra-small) / 1.6
      var(--body-font-family-secondary);
  }
  &:hover {
    --cp-actions-opacity: 1;
  }
}
.property {
  --property-name-text-transform: none;
}
.property-total {
  margin: var(--spacer-extra-big) 0 0 0;
  --property-name-font: 500 var(--font-size-extra-big) / 1.6
    var(--body-font-family-secondary);
  --property-name-color: var(--c-text);
  --property-value-font: 500 var(--font-size-extra-big) / 1.6
    var(--body-font-family-secondary);
}
.promo-code {
  &__button {
    --button-padding: 0;
    --button-text-decoration: none;
    --button-text-transform: uppercase;
    --button-font-size: var(--font-size-big);
  }
  &__input {
    margin: var(--spacer-big) 0;
  }
}
.characteristic {
  margin: 0 0 var(--spacer-big) 0;
  &:last-child {
    margin: 0;
  }
}
</style>

<template>
  <div class="sw-add-product-review">
    <div v-if="!wasReviewSent">
      <SfHeading :title="$t('Leave a review!')" :level="3" />
      <SfHeading
        :title="$t('Share your experiences with other customers.')"
        :level="4"
      />
      <form
        v-if="isLoggedIn"
        action=""
        class="add-review-form"
        @submit.prevent="submitForm"
      >
        <div class="input-group">
          <SwRating
            :score="Number.parseInt(reviewRating)"
            :max="5"
            :hover-effect="true"
            @click="setCurrentRating"
          />
          <SwInput
            v-model="reviewTitle"
            type="text"
            label="Title"
            name="title"
            error-message="Title is required"
            class="small input"
          />

          <textarea
            v-model="reviewContent"
            name="desc"
            class="description-field"
            placeholder="Write review"
          >
          </textarea>
        </div>

        <SwErrorsList :list="errorMessages" />

        <SwButton class="send button">
          {{ !isSending ? $t("Add review") : $t("Sending...") }}
        </SwButton>
      </form>

      <SwButton
        v-if="!isLoggedIn"
        class="login button"
        @click="switchLoginModalState"
      >
        {{ $t("Log in") }}
      </SwButton>
    </div>
    <div v-else>
      <SfHeading
        :title="$t('Your review has been sent. Thank you!')"
        :level="3"
      />
    </div>
    <SfDivider />
  </div>
</template>

<script>
import { SfHeading, SfDivider } from "@storefront-ui/vue"
import { addProductReview } from "@shopware-pwa/shopware-6-client"
import { getMessagesFromErrorsArray } from "@shopware-pwa/helpers"
import SwButton from "@/components/atoms/SwButton"
import SwInput from "@/components/atoms/SwInput"
import SwErrorsList from "@/components/SwErrorsList"
import { ref, computed } from "@vue/composition-api"
import {
  useUser,
  useUIState,
  getApplicationContext,
} from "@shopware-pwa/composables"

export default {
  name: "SwAddProductReview",
  components: {
    SwInput,
    SwButton,
    SwErrorsList,
    SfHeading,
    SfDivider,
  },
  props: {
    productId: {
      type: String,
      required: true,
    },
  },
  setup({ productId }, { root }) {
    const { isLoggedIn } = useUser(root)
    const { apiInstance } = getApplicationContext(root, "SwAddProductReview")
    const { switchState: switchLoginModalState } = useUIState(
      root,
      "LOGIN_MODAL_STATE"
    )

    const errorMessages = ref([])
    const reviewTitle = ref(null)
    const reviewContent = ref(null)
    const reviewRating = ref(0)
    const wasReviewSent = ref(false)
    const isSending = ref(false)
    const reviewRequestData = computed(() => ({
      title: reviewTitle.value,
      content: reviewContent.value,
      points: reviewRating.value,
    }))
    const setCurrentRating = (rating) => {
      reviewRating.value = rating
    }
    const submitForm = async () => {
      isSending.value = true
      try {
        console.warn("submiting", reviewRequestData.value)
        await addProductReview(productId, reviewRequestData.value, apiInstance)
        wasReviewSent.value = true
      } catch (error) {
        console.error("[SwAddProductReview][submitForm]: ", error)
        errorMessages.value = getMessagesFromErrorsArray(error.message)
      }
      isSending.value = false
    }

    return {
      isLoggedIn,
      switchLoginModalState,
      reviewTitle,
      reviewContent,
      reviewRating,
      errorMessages,
      submitForm,
      setCurrentRating,
      wasReviewSent,
      isSending,
    }
  },
}
</script>

<style lang="scss" scoped>
@import "@/assets/scss/forms";

.sw-add-product-review {
  width: 100%;
}

.add-review-form {
  align-items: center;
  display: flex;
  flex-direction: column;
  justify-content: center;

  .button {
    &.send {
      width: 100%;
      margin-top: var(--spacer-xl);
    }
  }

  .input-group {
    width: 100%;
  }

  .description-field {
    border: 1px solid var(--c-light);
    box-sizing: border-box;
    font-size: var(--font-lg);
    margin-top: var(--spacer-base);
    min-height: 200px;
    padding: var(--spacer-base);
    width: 100%;

    &:focus {
      outline-color: var(--_c-green-primary);
    }
  }
}

.login.button {
  width: 100%;
  margin-top: var(--spacer-base);
}

.sw-rating {
  margin-bottom: var(--spacer-base);
  margin-top: var(--spacer-base);
}
</style>

<template>

  <core-modal
    :title="currentTitle"
    @cancel="closeModal"
  >
    <div class="contents">
      <status
        v-if="showStatus"
        :status="status"
        @close="handleClose"
      />
      <phone-number-form
        v-else
        @submit="submitTokenRequest"
        @close="closeModal"
        :disabled="disableForms"
        :phoneLookupFailed="phoneLookupFailed"
      />
    </div>
  </core-modal>

</template>


<script>

  import coreModal from 'kolibri.coreVue.components.coreModal';
  import status from './status';
  import phoneNumberForm from './phone-number-form';
  import { createResetToken } from '../../state/resetPasswordActions';
  import { RequestTokenStates as STATES } from '../../constants';

  export default {
    name: 'resetPasswordModal',
    components: {
      coreModal,
      phoneNumberForm,
      status,
    },
    data() {
      return {
        disableForms: false,
        status: STATES.ENTER_PHONE_NUMBER,
      };
    },
    computed: {
      currentTitle() {
        switch (this.status) {
          case STATES.MESSAGE_SENT:
            return this.$tr('messageSent');
          case STATES.SMS_SERVICE_ERROR:
            return this.$tr('smsServiceError');
          default:
            return this.$tr('resetPassword');
        }
      },
      showStatus() {
        return this.status === STATES.MESSAGE_SENT || this.status === STATES.SMS_SERVICE_ERROR;
      },
      phoneLookupFailed() {
        return this.status === STATES.ACCOUNT_NOT_FOUND;
      },
    },
    methods: {
      submitTokenRequest(phoneNumber) {
        this.disableForms = true;
        this.createResetToken({ phoneNumber })
          .then(() => {
            this.status = STATES.MESSAGE_SENT;
          })
          .catch(err => {
            const { code } = err.status;
            if (code === 400) {
              this.status = STATES.ACCOUNT_NOT_FOUND;
            } else {
              this.status = STATES.SMS_SERVICE_ERROR;
            }
          })
          .then(() => {
            this.disableForms = false;
          });
      },
      resetState() {
        this.status = STATES.ENTER_PHONE_NUMBER;
      },
      closeModal() {
        // guard against closing until 'X' button can be removed
        if (!this.disableForms) {
          this.$emit('close');
        }
      },
      handleClose() {
        if (this.status === STATES.ACCOUNT_NOT_FOUND) {
          this.status = STATES.ENTER_PHONE_NUMBER;
        } else {
          this.closeModal();
        }
      },
    },
    vuex: {
      actions: {
        createResetToken,
      },
    },
    $trs: {
      accountNotFound: 'Account not found',
      messageSent: 'Message sent',
      resetPassword: 'Reset password',
      smsServiceError: 'SMS service error',
    },
  };

</script>


<style lang="stylus" scoped>

  .contents
    margin: 1em 0

</style>

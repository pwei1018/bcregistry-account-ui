<template>
  <v-container class="view-container mt-10">
    <v-row justify="center">
      <v-col cols="12" sm="8" md="8" class="text-center pb-0">
        <v-icon large color="info" class="font-weight-bold">mdi-alert-circle-outline</v-icon>
        <h1 class="mt-2">Please Update Your Account Information</h1>
        <p class="text-center mt-8"
           v-html="$t('updateAccountInformation' , {'name': currentOrganization.name })" />
      </v-col>
    </v-row>
    <v-row justify="center">
      <v-col cols="12" sm="9" md="8" class="text-center">
        <v-card flat class="my-4 justify-space-between pa-8">
          <v-card-title>
            <h2>Account Information</h2>
          </v-card-title>
          <v-card-text>
            <p style="text-align: justify">
              Do you want your account: <span class="font-weight-bold">{{ currentOrganization.name }}</span> associated
              with your personal name or business name?
            </p>
            <v-form ref="accountInformationForm" data-test="account-information-form">

              <v-alert type="error" class="mb-6" v-show="errorMessage">
                {{ errorMessage }}
              </v-alert>

              <v-radio-group
                row
                v-model="isBusinessAccount"
              >
                <v-row justify="space-between">
                  <v-col xs="12" md="12" class="business-radio">
                    <v-radio
                      label="Individual Person Name"
                      :key="false"
                      :value="false"
                      data-test="radio-individual-account-type"
                      class="px-4 py-5"
                    ></v-radio>
                    <v-radio
                      label="Business Name"
                      :key="true"
                      :value="true"
                      data-test="radio-business-account-type"
                      class="px-4 py-5"
                    ></v-radio>
                  </v-col>
                </v-row>
              </v-radio-group>

              <div v-if="isBusinessAccount">
                <v-text-field
                  filled
                  label="Account Name"
                  disabled
                  v-model="currentOrganization.name"
                >
                </v-text-field>
                <v-expand-transition class="branch-detail">
                  <v-text-field
                    filled
                    label="Branch/Division (Optional)"
                    v-model.trim="branchName"
                    data-test="input-branch-name"
                  />
                </v-expand-transition>
                <AccountBusinessTypePicker
                  @valid="checkOrgBusinessTypeValid"
                  @update:org-business-type="updateOrgBusinessType">

                </AccountBusinessTypePicker>
              </div>

            </v-form>
          </v-card-text>

        </v-card>
      </v-col>
    </v-row>
    <v-row justify="center">
      <v-col cols="12" sm="8" md="6" class="text-center">
        <v-btn large color="primary" :disabled="!canSubmit" data-test="goto-create-account-button" @click="submit()">
          Submit
        </v-btn>
      </v-col>
    </v-row>
  </v-container>
</template>

<script lang="ts">
import { Component, Vue, Watch } from 'vue-property-decorator'
import { CreateRequestBody, OrgBusinessType, Organization } from '@/models/Organization'
import AccountBusinessTypePicker from '@/components/auth/common/AccountBusinessTypePicker.vue'
import { Pages } from '@/util/constants'
import { namespace } from 'vuex-class'

const OrgModule = namespace('org')

@Component({
  components: { AccountBusinessTypePicker }
})
export default class UpdateAccountView extends Vue {
  @OrgModule.State('currentOrganization') private currentOrganization!: Organization
  @OrgModule.Action('updateOrg') private updateOrg!: (requestBody: CreateRequestBody) => Promise<Organization>
  private errorMessage: string = ''
  private isBusinessAccount = null
  private canSubmit = false
  private branchName = ''
  private orgBusinessType: OrgBusinessType = null

  private async mounted () {
    this.branchName = this.currentOrganization?.branchName
  }

  @Watch('isBusinessAccount')
  async onOrgBusinessTypeChange () {
    this.canSubmit = this.isBusinessAccount === false
  }

  private checkOrgBusinessTypeValid (isValid) {
    this.canSubmit = isValid ? true : this.canSubmit
  }

  private async submit () {
    let createRequestBody: CreateRequestBody = {
      isBusinessAccount: this.isBusinessAccount
    }
    if (this.isBusinessAccount) {
      createRequestBody.branchName = this.branchName
      createRequestBody.businessSize = this.orgBusinessType.businessSize
      createRequestBody.businessType = this.orgBusinessType.businessType
    }
    try {
      await this.updateOrg(createRequestBody)
      await this.$router.push(`/${Pages.HOME}`)
    } catch (err) {
      switch (err.response.status) {
        case 409:
          this.errorMessage =
            'An account with this branch name already exists. Try a different branch name.'
          break
        case 400:
          this.errorMessage = 'Invalid account name'
          break
        default:
          this.errorMessage =
            'An error occurred while attempting to update your account.'
      }
    }
  }

  private updateOrgBusinessType (orgBusinessType: OrgBusinessType) {
    this.orgBusinessType = orgBusinessType
  }
}
</script>

<style lang="scss" scoped>
@import '$assets/scss/theme.scss';

.view-container {
  padding: 0 !important;
}

.business-radio {
  display: flex;

  .v-radio {
    padding: 10px;
    background-color: rgba(0, 0, 0, .06);
    min-width: 50%;
    border: 1px rgba(0, 0, 0, .06) !important;
  }

  .v-radio.theme--light.v-item--active {
    border: 1px solid var(--v-primary-base) !important;
    background-color: $BCgovInputBG !important;
  }
}
</style>

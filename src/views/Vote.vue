<template>
<div>
  <section class="slice slice-lg vh-100 bg-gradient-primary overflow-hidden" data-offset-top="#header-main" v-if="proposal === undefined">
    <div class="container h-100 d-flex align-items-center position-relative zindex-100">
      <div class="col">
        <div class="row justify-content-center">
          <div class="col-lg-7 text-center">
            <h6 class="h1 mb-5 font-weight-400 text-white">{{$t('cantFindProposal')}}</h6>
              <router-link class="btn btn-white btn-icon rounded-pill hover-translate-y-n3" to="/">
                <span class="btn-inner--icon">
                  <i class="fas fa-home"></i>
                </span>
                <span class="btn-inner--text">{{$t('returnHome')}}</span>
              </router-link>
          </div>
        </div>
      </div>
    </div>
  </section>
  <div v-if="proposal">
   <section class="slice bg-gradient-primary vh-100 pt-5">
    <div class="container py-5 position-relative zindex-100">
      <div class="row row-grid justify-content-around align-items-center">
        <div class="col-lg-5">
          <div class="text-center text-lg-left">
            <span class="badge badge-soft-primary badge-pill">{{$t('vote.steemOn')}}</span>
            <h1 class="text-white mt-4 mb-3">{{$t('vote.steemProposalsTitle')}}</h1>
            <!-- Buttons -->
            <div class="mt-6">
              <a class="btn btn-app-store hover-translate-y-n3 mr-lg-4 mb-4 text-left" @click="$bvModal.show('vote')">
                <i><img class="icon-small ml-1" src="../assets/img/random/keychain.png"/></i>
                <span class="btn-inner--text">{{$t('vote.supportWith')}}</span>
                <span class="btn-inner--brand">Keychain</span>
              </a>
              <a class="btn btn-app-store hover-translate-y-n3 mb-4 text-left" @click="steemconnectVote(true)">
                <i><img class="icon-small ml-1" src="../assets/img/random/steemconnect.png"/></i>
                <span class="btn-inner--text">{{$t('vote.supportWith')}}</span>
                <span class="btn-inner--brand">Steemconnect</span>
              </a>
               <b-modal id="vote" :title="`${$t('vote.supportingProposal')}`" centered hide-footer>
                 <b-form @submit.prevent="keychainVote(user, voteStatus)">
                  <b-form-group
                    id="user_group"
                    :label="`${$t('keychain.inputLabel')}`"
                    label-for="user">
                    <b-form-input
                      id="user"
                      v-model="user"
                      type="text"
                      required
                      :placeholder="`${$t('keychain.placeholder')}`">
                    </b-form-input>
                  </b-form-group>
                  <b-form-group>
                    <div class="mb-2">{{$t('keychain.voteLabel1')}}</div>
                    <b-form-checkbox v-model="voteStatus" name="vote-button" switch>
                      <b>{{ voteStatus ? $t('keychain.voteApprove') : $t('keychain.voteRemove') }}</b>
                    </b-form-checkbox>
                  </b-form-group>
                  <b-form-group>
                    <div class="mb-2">{{$t('keychain.voteLabel3')}}</div>
                    <button class="btn-block btn btn-light" type="submit" variant="light">{{$t('keychain.voteWithLabel')}} <img class="icon-small ml-1" src="../assets/img/random/keychain2.png"/></button>
                  </b-form-group>
                </b-form>
              </b-modal>
            </div>
            <div class="text-white">
              <a href="https://signup.steemit.com" target="_blank" class="text-white"><small>{{$t('vote.steemAccount')}}</small></a>
            </div>
          </div>
        </div>
        <div class="col-lg-5">
          <div class="card bg-section-secondary mt-6 mb-0 py-3 px-4 shadow-lg">
            <div class="card-body">
              <h5 class="heading h5 mb-3">{{proposal.subject}}</h5>
              <div class="mb-3" style="cursor:pointer" @click="loadVoters()"><strong class="h3">{{totalVotersByProposalId(Number(id)) || 0}}</strong><span> {{$t('vote.supportersLabel')}}</span></div>
              <b-progress id="tooltip1" height="2rem" variant="primary" :max="100">
                <b-progress-bar :class="totalValue() < 10 ? 'text-dark pl-2' : 'text-white pl-2'" :value="totalValue() || 0" :label="`${totalValue() || 0}%`"></b-progress-bar>
              </b-progress>
              <b-tooltip target="tooltip1" triggers="hover">
                <p v-if="returningProposal">{{$t('vote.totalVotesValue')}} {{totalProposalSP(proposal.id) | numeric3}} SP</p>
              </b-tooltip>
              <ul class="list-unstyled mt-4">
                <li class="py-2">
                  <div class="d-flex align-items-center">
                    <div>
                      <div class="icon icon-shape icon-success icon-sm rounded-circle mr-3">
                        <i class="fas fa-book"></i>
                      </div>
                    </div>
                    <div>
                      <span class="h6 mb-0"><strong>{{$t('vote.details')}} </strong> <a :href="`https://steemit.com/@${proposal.creator}/${proposal.permlink}`" target="_blank">{{$t('vote.steemitPost')}}</a></span>
                    </div>
                  </div>
                </li>
                <li class="py-2">
                  <div class="d-flex align-items-center">
                    <div class="icon icon-shape icon-warning icon-sm rounded-circle mr-3">
                      <i class="fas fa-wallet"></i>
                    </div>
                    <span class="h6 mb-0"><strong>{{$t('vote.requestedFunding')}}: </strong> {{totalRequested(proposal) | numeric3}} SBD</span>
                  </div>
                </li>
                 <li class="py-2">
                  <div class="d-flex align-items-center">
                    <div class="icon icon-shape icon-primary icon-sm rounded-circle mr-3">
                      <i class="fas fa-calendar"></i>
                    </div>
                    <span class="h6 mb-0"><strong>{{duration(proposal) | numeric3}} {{$t('common.days')}}</strong> ({{proposal.start_date | dateFilter}} - {{proposal.end_date | dateFilter}})</span>
                  </div>
                </li>
                <li class="py-2">
                  <div class="d-flex align-items-center">
                    <a :href="`https://steemit.com/@${proposal.creator}`" target="_blank" class="avatar avatar-sm rounded-circle mr-3">
                      <img alt="Image placeholder" :src="`https://steemitimages.com/u/${proposal.creator}/avatar`">
                    </a>
                    <span class="h6 mb-0"><strong>{{$t('vote.createdBy')}}</strong> @<a target="_blank" :href="`https://steemit.com/@${proposal.creator}`">{{proposal.creator}}</a></span>
                  </div>
                </li>
              </ul>
            </div>
          </div>
        </div>
      </div>
    </div>
     <!-- Voters modal -->
  <b-modal size="md" scrollable ref="modal-voters" :title="`${$t('proposals.proposalVoters')}`" centered hide-footer>
    <div class="row">
      <div class="col-12 d-flex justify-content-center" v-if="!accounts.length">
        <b-spinner label="Spinning"></b-spinner>
        <span class="ml-3">{{$t('common.loadingVotes')}}</span>
      </div>
    </div>
    <div class="row">
      <div class="col-12 d-flex justify-content-center">
        <b-list-group v-if="accounts.length">
          <h5>{{$t('vote.supportedByCommunity')}}:</h5>
          <b-list-group-item v-for="(voter, index) in votersByProposalId(Number(id))" :key="index" class="d-flex justify-content-between align-items-center">
            <div class="avatar rounded-circle">
              <a :href="`https://steemit.com/@${voter.voter}`" target="_blank">
                <img :src="`https://steemitimages.com/u/${voter.voter}/avatar`" />
              </a>
            </div>
            <a class="text-dark" :href="`https://steemit.com/@${voter.voter}`" target="_blank">@{{voter.voter}}</a> 
            <b-badge v-if="voter.proxyAccount === ''" variant="light" class="p-2">{{voter.sp | numeric3}} SP + <br/>{{voter.proxySP | numeric3}} SP {{$t('common.proxy')}}</b-badge>
            <b-badge v-else variant="light" class="p-2">0 SP <br/> ({{voter.totalSP | numeric3}} SP {{$t('common.proxied')}} @{{voter.proxyAccount}})</b-badge>
          </b-list-group-item>
         </b-list-group>
       </div>
     </div>
   </b-modal>
  </section>
  </div>
</div>
</template>

<script>
import { mapState , mapGetters } from 'vuex'
export default {
  name: 'ProposalVote',
  props: ['id'],
  computed: {
    ...mapState(['proposal', 'voters', 'accounts']),
    ...mapGetters({
      duration: 'totalProposalDuration',
      totalRequested: 'totalRequested2',
      returningProposal: 'returningProposal',
      votersByProposalId: 'votersByProposalId',
      totalVotersByProposalId: 'totalVotersByProposalId',
      totalProposalSP: 'totalProposalSP',
      vestsToSP: 'vestsToSP'
    })
  },
  data () {
    return {
      user: '',
      voteStatus: true
    }
  },
  methods: {
    keychainVote (user, approve) {
      if (window.steem_keychain && user !== '') {
        steem_keychain.requestBroadcast(user, [["update_proposal_votes", {"voter":user,"proposal_ids":[`${this.id}`],"approve":`${approve}`}]], 'Active', function (response) {
          if (response.success) {
            return response
          } else {
            return response.success
          }
        })
      } else {
        return []
      }
    },
    steemconnectVote (approve) {
      window.open(`https://beta.steemconnect.com/sign/update-proposal-votes?proposal_ids=[${this.id}]&approve=${approve}`)
    },
    fetchProposalById () {
      this.$store.dispatch('fetchProposalById', Number(this.id))
    },
    totalValue () {
      if (this.proposal && this.returningProposal && this.totalProposalSP(Number(this.id))) {
        let proposalSP = this.totalProposalSP(Number(this.id))
        let returningSP = this.vestsToSP(this.returningProposal.total_votes)
        let value = Number(proposalSP / returningSP * 100).toFixed(2)
        return Number(value)
      }
    },
    loadVoters () {
      this.$refs['modal-voters'].show()
    }
  },
  created () {
    this.fetchProposalById()
  }
}
</script>

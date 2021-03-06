<template>
  <div class="display-details">
    <div v-if="userData" class="user-details">
      <div class="row-name">
        <button
          v-if="ownProfile"
          class="button-submit button-submit-reverse"
          style="margin-bottom:1rem;"
          @click="profileEdit">
          Edit Profile
        </button>
        <h3 class="display-name">
          {{ displayName }} {{ displayLastName }}
        </h3>
      </div>
      <p class="display-bio text-muted">
        {{ displayBio }}
      </p>
      <p class="display-website" v-if="userData.website">
        <a
          :href="userData.website | externalUrl"
          target="_blank"
          rel="external nofollow">
          {{ userData.website | host }}
        </a>
      </p>
      <p class="display-website" v-if="userData.twitter">
        <a :href="`https://twitter.com/${userData.twitter}`" target="_blank">
        @{{ userData.twitter }}
        </a>
      </p>
    </div>
    <div class="crop-image">
      <img class="profile-img" :src="errorImg || avatarUrl" @error="imgOnError">
    </div>
  </div>
</template>

<script>
  import modal from '@/components/ModalComponent'
  import { mapState, mapActions, mapGetters } from "vuex";

  export default {
    name: 'profile-details',
    props: {
      userData: {
        type: Object,
        default: function () {
          return {
            _id: '',
            username: '',
            avatarUrl: '',
            bio: '',
            name: '',
            lastName: '',
            website: '',
            twitter: ''
          }
        }
      },
      ownProfile: {
        type: Boolean,
        default: false
      }
    },
    components: {
      modal,
    },
    data () {
      return {
        isModalVisible: false,
        userTopics: [],
        searchTopic: '',
        checkedTopics: [],
        topics:[],
        isOpen: false,
        modalTopics: [],
        errorImg: ''
      }
    },
    mounted () {
      document.addEventListener('click', this.handleClickOutside)
    },
    destroyed() {
      document.removeEventListener('click', this.handleClickOutside);
    },
    computed: {
      ...mapGetters(["isLoggedIn"]),
      ...mapState({
        displayName () {
          return this.userData.name || this.userData.username
        },
        displayLastName () {
          return this.userData.lastName || ''
        },
        displayBio () {
          return this.userData.bio || `${this.displayName} is still writing their biography`
        },
        avatarUrl(state) {
          return (this.userData) ? this.userData.avatarUrl || state.placeholderAvatar : state.placeholderAvatar
        },
        publicLink (state) {
          if (!this.userData || !this.userData.name) return null;
          return `/profile/${this.userData._id}`
        }
      })
    },
    methods: {
      ...mapActions(['getUserTopics','getSearchedTopics','addTopicToUser', 'fetchPublicProfileData']),
      logoutHandler () {
        this.$auth.logout()
        this.$router.replace('/')
      },
      setResult(item) {
        const topic = _.find(this.modalTopics, (x) => ( x._id === item._id ))
        if (!topic) {
          this.modalTopics.push(item);
          this.checkedTopics.push(item._id);
        }
      },
      profileEdit() {
        this.$router.push('/edit-profile')
      },
      debounceSearchRequest: _.debounce(function () {
        this.getSearchedTopics(this.searchTopic)
      }, 500),
      handleClickOutside(evt) {
        if (this.$el.contains(evt.target)) {
          this.isOpen = false;
        }
      },
      goTo(slug){
        this.$router.push(`/topics/${slug}`)
      },
      onChange() {
        this.isOpen = true;
        this.getSuggestedTopics()
      },
      getSuggestedTopics() {
        if(this.searchTopic !== '') {
          this.debounceSearchRequest()
        }
      },
      imgOnError(event) {
        this.errorImg = 'https://s3-us-west-2.amazonaws.com/sd-profile-pictures/profile-icon-9.png'
      },
      showModal() {
        this.checkedTopics = []
        this.userTopics.map((item) => {
          this.modalTopics.push(item)
        })
        this.$store.state.topics.user.map((item) => {
          this.checkedTopics.push(item._id)
        })
        this.isModalVisible = true;
        $("body").addClass("modal-open")
      },
      closeModal() {
        this.modalTopics = []
        this.isModalVisible = false
        this.searchTopic = ''
        $("body").removeClass("modal-open")
      },
      getTopics() {
        this.getUserTopics()
      },
      selectTopicsToUser() {
        const selectedTopics = []
        this.checkedTopics.map((id) => {
          selectedTopics.push(id)
        })
        const userTopics = {
          userId: this.$store.state.me._id,
          topics: selectedTopics
        }
        this.addTopicToUser(userTopics).then(this.getTopics).then(this.closeModal)
      },
      filterItems: function() {
        const app = this
        this.userTopics = this.$store.state.topics.user
        this.topics = this.$store.state.topics.searchedAllTopics
        if(this.topics !== null){
          return this.topics
        }
      }
    }
  }
</script>

<style scoped lang="stylus">
@import '../../css/variables'
.display-details
  display flex
  justify-content space-between
  margin-left auto
  margin-right auto
  padding 10px

.row-name
  display flex
  flex-direction column
  align-items flex-start
  max-width 400px

  .button-submit
    max-height 30px
    min-width 90px

  .button-submit-reverse
    background-color inherit
    border 1px solid primary-color
    padding 5px 10px
    color primary-color

@media (max-width 450px)
  .display-details
    flex-direction column-reverse
    justify-content center

  .user-details
    margin 15px 0
    text-align center !important

    .display-name
      margin 0 auto !important

  .crop-image
    margin-left auto !important
    margin auto

  .button-submit
    margin-left auto !important
    margin 10px auto

  .row-name
    flex-direction column-reverse
    justify-content center

@media (max-width 750px)
  .user-topics
    overflow auto
    white-space nowrap
    flex-wrap nowrap!important

.user-topics-header
  margin 20px 0
  display flex
  align-items center

  .user-topics
    display flex
    align-items center
    max-width 100%
    flex-wrap wrap

    .topics
      background-color primary-color
      color white
      margin 2px 0
      margin-right 5px
      padding 5px
      border-radius 5px
      cursor pointer

.btn-link
  color #ccc
  background none
  border none
  outline none
  cursor pointer
  font-weight 500
  padding-left 15px

.profile
  padding-top 2rem
  justify-content center

.wrapper
  margin-left auto
  margin-right auto
  padding 10px

.user-details
  text-align left

  .display-name
    font-weight 600
    margin 0

  .display-website a
    text-decoration none
    color primary-color

    &:hover
      font-weight bold

  .text-muted
    padding-top 15px
    font-weight 400
    max-width 400px

.crop-image
  background-position center
  background-size: cover
  background-repeat no-repeat
  border-radius 50%
  min-width 100px
  width 100px
  height 100px
  margin-left 15px
  overflow hidden
  transition all .5s ease

  &:hover
    width 120px
    height 120px

.profile-img
  width inherit

.edit-link
  padding 10px
  text-align left
  margin 0 auto

  a
    color primary-color

.autocomplete
  max-width 500px
  margin auto

  .absolute
    background white
    z-index 1000
    height auto!important
    position absolute

.search-label
  width 100%
  cursor pointer

  &:hover
    color primary-color

body.modal-open
  overflow hidden

.btn-modal
  background-color primary-color
  color white
  margin-right 5px
  padding 5px
  border-radius 0 5px 5px 0
  border 1px solid #c4c4c4
  cursor pointer
  outline none

.btn-modal-secondary
  background-color white
  color black
  margin-right 5px
  padding 5px
  border none
  cursor pointer
  outline none

h2
  margin 0
  color black
  font-size 1.5rem

.post-topics-header
  margin 15px 20px
  display flex
  align-items center

  .post-topics
    display flex
    align-items center
    max-width 70%
    flex-wrap wrap

    .topics
      background-color primary-color
      color white
      margin 2px 0
      margin-right 5px
      padding 5px
      border-radius 5px
      border 1px solid #c4c4c4

.search-bar
  justify-content center
  flex 1
  display flex
  align-items center

  input
    max-width 500px
    flex 1
    border-radius 5px
    padding 5px
    font-weight 100
    color #C4C4C4
    border 1px solid #ccc

    &:focus
      outline none
      color #495057
      background-color #fff
      border-color: #edeaff
      outline 0
      box-shadow 0 0 0 0.2rem rgba(133, 106, 255, 0.25)

.container
  width auto
  display flex
  align-items center
  position relative
  padding-left 35px
  margin-bottom 12px
  margin-top 12px
  cursor pointer
  font-size 16px
  -webkit-user-select none
  -moz-user-select none
  -ms-user-select none

  input
    position absolute
    opacity 0
    cursor pointer
    height 0
    width 0

  .checkmark
    position absolute
    left 0
    height 20px
    width 20px
    background-color #eee

.container:hover input ~ .checkmark
  background-color #ccc

.container input:checked ~ .checkmark
  background-color primary-color

.checkmark:after
  content ""
  position absolute
  display none

.container input:checked ~ .checkmark:after
  display block

.container .checkmark:after
  left 7px
  top 3px
  width 6px
  height 12px
  border 2px solid white
  border-width 0 3px 3px 0
  -webkit-transform rotate(45deg)
  -ms-transform rotate(45deg)
  transform rotate(45deg)

.no-topic
  display flex
  flex-direction column
  overflow auto
  overflow-x hidden
  width 100%
  height 100%
  max-height 250px
  max-width 500px
  margin auto
  padding 0
  padding 5px

.popular-topics
  display flex
  flex-direction column
  overflow auto
  overflow-x hidden
  width 100%
  height 100%
  max-height 250px
  max-width 500px
  margin auto
  padding 0
  border-radius 5px
  border 1px solid #c4c4c4
  padding 10px
  list-style none

.popular-topic
  justify-content center
  text-align left

.popular-topics::-webkit-scrollbar
  width 5px

.popular-topics::-webkit-scrollbar-track
  background #f1f1f1

.popular-topics::-webkit-scrollbar-thumb
  background #d0c6ff

.popular-topics::-webkit-scrollbar-thumb:hover
  background #555

.public-link
  text-decoration none
  color primary-color

  &:hover
    font-weight bold

</style>

<template>
  <!-- 
    * 작성자 : 서울2반 4팀 황윤호
    * 내용 : ui수정
    * 생성일자 : 2021-02-15
    * 최종수정일자 : 2021-02-19
  -->
  <v-app>
    <v-toolbar id="navi_shadow">
      <!-- 뒤로가기 누르면 검색페이지로 이동 -->
      <v-icon @click="backPage">mdi-keyboard-backspace</v-icon>
      <v-toolbar-title class="flex text-left" font-family>사용자 조회</v-toolbar-title>
    </v-toolbar>
    <div class="profile_info">
      <!-- 사용자 프로필 이미지 -->
      <div class="user_info">
        <!-- 이미지 없을 때 -->
        <v-avatar class="user_avatar" size="120px">
          <img
            v-if="$route.query.profileDefaultPic === null"
            src="@/assets/images/mindwiki_logo-03.png"
            alt="John"
          />
          <!-- 이미지 있을 때 -->
          <img v-else :src="$route.query.profileDefaultPic" alt="John" />
        </v-avatar>

        <!-- 사용자 계정 정보 -->
        <div class="user_nickname text-center p-t-22 p-b-3">
          <span>{{ $route.query.nickName }}</span>
        </div>
        <div class="txt1 text-center p-t-2 p-b-20">
          <span>{{ $route.query.email }}</span>
        </div>

        <!-- 팔로우 하기 -->
        <w-card>
          <w-card-body>
            <div class="user-num">
              <w-button v-if="this.show === false" class="profile_button" @click="userfollow()"
                >팔로우 하기</w-button
              >
              <w-button
                v-if="this.show === true"
                class="profile_button"
                @click="deletefollower($route.query.email)"
                >팔로우 취소</w-button
              >
            </div>
          </w-card-body>
        </w-card>

        <!-- 내가 팔로우한 사람의 MIND목록 -->
        <div class="mymind_list p-t-60 p-b-20">
          <div class="txt2 p-t-15 p-b-15">
            <span>{{ $route.query.nickName }}님의 MIND</span>
          </div>
          <mind-list :mindlist="minditems"> </mind-list>
        </div>
      </div>
    </div>
  </v-app>
</template>

<script>
import { mapGetters } from 'vuex';
import MindList from './MindList.vue';
export default {
  components: { MindList },
  name: 'UserProfile',
  computed: {
    ...mapGetters(['followingData']),
    ...mapGetters(['followingMindData']),
    // default 이미지 설정
    creatorImage() {
      return this.imageError ? this.defaultImage : 'creator-image.jpg';
    },
  },
  data() {
    return {
      user: {
        files: '',
        usernickName: '',
        useremail: '',
      },
      followeremail: '',
      message: '',
      items: [],
      minditems: [],
      followerlist: [],
      show: false,
      profile: '',
      defaultImage: require('@/assets/images/mindwiki_logo-color.png'),
      imageError: false,
    };
  },
  methods: {
    // 팔로우 하기
    userfollow() {
      const form = new FormData();
      form.append('jwt', this.$store.getters.getJWT);
      form.append('followeremail', this.$route.query.email);
      this.$store.dispatch('userFollow', form).then(() => {
        this.message = this.$store.getters.getMessage;
        this.show = !this.show;
      });
    },
    // 팔로우한 유저의 마인드 조회
    readfollowingminddata() {
      const form = new FormData();
      form.append('jwt', this.$store.getters.getJWT);
      form.append('followeremail', this.$route.query.email);
      this.$store.dispatch('readfollowingMindData', form).then(() => {
        this.minditems = this.$store.getters.followingMindData;
      });
    },
    // 검색페이지로 이동
    backPage() {
      this.$router.push('/main');
    },
    // 팔로잉 확인
    readfollowing() {
      this.$store.dispatch('readFollowing', this.$store.getters.getJWT).then(() => {
        this.items = this.$store.getters.followingData;
        for (var i = 0; i < this.items.length; i++) {
          if (this.items[i].followerEmail === this.$route.query.email) {
            this.show = true;
            break;
          }
          this.show = false;
        }
      });
    },
    // 팔로잉 목록에서 제거
    deletefollower(followeremail) {
      const form = new FormData();
      form.append('jwt', this.$store.getters.getJWT);
      form.append('followeremail', followeremail);
      this.$store.dispatch('deleteFollower', form).then(() => {
        this.show = false;
      });
    },
  },
  mounted() {},
  created: function() {
    this.readfollowingminddata();
    // 프로필 정보 받아오기
    let form = new FormData();
    form.append('jwt', this.$store.getters.getJWT);
    this.$store.dispatch('myProfile', form).then(() => {
      // 응답 결과
      this.message = this.$store.getters.message;
      this.profile = this.$store.getters.profile;
      if (this.message === 'FAIL')
        this.showAlert('세션이 만료되었습니다. 다시 로그인 해 주세요.', '프로필 수정');
      else {
        this.user.usernickName = this.profile.nickName;
        this.followerEmail = this.profile.email;
        this.user.files = this.profile.profileDefaultPic;
        this.readfollowing();
      }
    });
  },
};
</script>

<style scoped>
.profile_info {
  background-image: url(../../../assets/images/profile/hero_mini.svg);
  min-height: auto;
  min-width: 360px;
  width: 100%;
  /* height: 100%; */
  position: relative;
  /* object-position: bottom; */
}
.user_info {
  padding: 20px;
}
/* 프로필 사진 */
.user_avatar {
  margin-top: 40px;
}
/* 계정에 해당하는 닉네임 */
.user_nickname {
  color: white;
  font-family: Poppins-Medium;
  font-size: 1.75rem;
}
/* 계정에 해당하는 이메일 */
.txt1 {
  color: white;
  font-family: Poppins-Medium;
}
.profile_button {
  margin-right: 10px;
  /* background-color: #a64bf4; */
}

/* 게시물 수 들어있는 card */
.user-num {
  display: inline-block;
  margin-left: 6px;
  margin-right: 12px;
  /* margin-top: 20px; */
}
/* 게시물, 스크랩, 팔로워, 팔로잉 title */
.card-title {
  font-size: 1.1rem !important;
  margin-bottom: 0px;
}
/* 게시물, 스크랩, 팔로워, 팔로잉 숫자 */
.card-description {
  text-align: center;
  /* margin: auto !important; */
}
.txt2 {
  text-align: left;
  margin-left: 20px;
  font-family: Poppins-Medium;
  font-size: 1.1rem !important;
}
.card-body {
  padding: 1rem;
}
@media (min-width: 320px) and (max-width: 480px) {
  .profile_info {
    background-image: url(../../../assets/images/profile/hero_mini.svg);

    min-height: auto;
    min-width: 360px;
    width: 100%;
    /* height: 100%; */
    position: relative;
    /* object-position: bottom; */
  }
  .user_info {
    max-width: 375px;
    margin: 0 auto;
  }
}

@media (min-width: 576px) {
  .profile_info {
    background-image: url(../../../assets/images/profile/hero_mini.svg);

    min-height: auto;
    min-width: 360px;
    width: 100%;
    /* height: 100%; */
    position: relative;
    /* object-position: bottom; */
  }
  .user_info {
    max-width: 540px;
    margin: 0 auto;
  }
}

@media (min-width: 768px) {
  .profile_info {
    background-image: url(../../../assets/images/profile/hero_mini.svg);

    min-height: auto;
    min-width: 360px;
    width: 100%;
    /* height: 100%; */
    position: relative;
    /* object-position: bottom; */
  }
  .user_info {
    max-width: 720px;
    margin: 0 auto;
  }
}

@media (min-width: 992px) {
  .profile_info {
    background-image: url(../../../assets/images/profile/hero_mini.svg);
    background-size: contain;

    min-height: auto;
    min-width: 360px;
    width: 100%;
    /* height: 100%; */
    position: relative;
    /* object-position: bottom; */
  }
  .user_info {
    max-width: 960px;
    margin: 0 auto;
  }
}

@media (min-width: 1200px) {
  .profile_info {
    background-image: url(../../../assets/images/profile/hero_mini.svg);
    background-size: contain;

    min-height: auto;
    min-width: 360px;
    width: 100%;
    /* height: 100%; */
    position: relative;
    /* object-position: bottom; */
  }
  .user_info {
    max-width: 1140px;
    margin: 0 auto;
  }
}
</style>

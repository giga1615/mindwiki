<template>
  <!-- 
    * 작성자 : 서울2반 4팀 황윤호
    * 내용 : 상세 마인드맵 노드 댓글 프로필 사진 조회
    * 생성일자 : 2021-02-09
    * 최종수정일자 : 2021-02-17
  -->
  <v-app>
    <div>
      <v-toolbar id="navi_shadow">
        <!-- 뒤로가기 누르면 해당하는 상세 마인드맵 페이지로 이동 -->
        <v-icon @click="backPage()">mdi-keyboard-backspace</v-icon>
        <v-spacer></v-spacer>
        <img src="@/assets/images/user/mindwiki_logo.png" alt="" height="23px" />
        <v-spacer></v-spacer>
      </v-toolbar>

      <!-- 댓글 검색 -->
      <v-toolbar flat>
        <v-toolbar-title>댓글 {{ this.items.length }}</v-toolbar-title>

        <v-spacer></v-spacer>

        <v-text-field
          type="searche"
          class="searched"
          placeholder="댓글내용 검색"
          outlined
          rounded
          hide-details="true"
          v-model="searche"
          @keypress.enter="onInputKeyword"
        >
          <template v-slot:append>
            <v-icon id="searchIcon" @click="onInputKeyword">mdi-magnify</v-icon>
          </template>
        </v-text-field>
      </v-toolbar>

      <v-divider id="divider"></v-divider>

      <!-- 댓글 목록 -->
      <v-card flat height="100%" class="comment_list">
        <v-list>
          <template v-for="(item, index) in searchHandler">
            <v-subheader class="d-flex justify-center" v-if="item.header" :key="item.header">
              {{ item.header }}
            </v-subheader>
            <v-divider v-else-if="item.divider" :key="index" :inset="item.inset"></v-divider>

            <v-list-item v-else :key="item.index">
              <!-- 댓글 이미지 첨부 -->
              <v-list-item-avatar>
                <!-- 프로필 사진에서 이미지 없을 때 default 이미지 -->
                <v-img
                  v-if="item.profileDefaultPic === null"
                  :src="creatorImage"
                  @error="imageError = true"
                ></v-img>
                <!-- 프로필에서 사진 등록하면 이미지 변경 -->
                <v-img v-else :src="item.profileDefaultPic"></v-img>
              </v-list-item-avatar>
              <!-- 프로필 사진 옆 댓글 작성자의 이메일 출력 -->
              <v-list-item-subtitle id="commentEmail" v-html="item.nickName"></v-list-item-subtitle>

              <v-list-item-title id="commentTitle" v-html="item.data"></v-list-item-title>

              <!-- 로그인 된 email과 댓글 작성 email 같을 때만 삭제 가능 -->
              <div v-bind:userId="userId" v-if="userId === item.email">
                <v-icon id="deleteBtn" @click="deletecomment(item.commentID)">mdi-trash-can</v-icon>
              </div>
            </v-list-item>
          </template>
        </v-list>
      </v-card>
    </div>

    <!-- 댓글 입력창 -->
    <v-footer id="footer" elevation="3">
      <!-- textarea : 엔터 누르면 줄바꿈 가능 / no-resize : 줄바꿔도 textarea 늘어나지 않음-->
      <v-textarea
        append-outer-icon="mdi-comment"
        @click:append-outer="createcomment"
        no-resize
        label="댓글 입력"
        rows="1"
        color="purple"
        id="write"
        ref="write"
        rounded
        class="search"
        hide-details="true"
        v-model="write"
      >
      </v-textarea>
    </v-footer>
  </v-app>
</template>

<script>
import { mapGetters } from 'vuex';
export default {
  name: 'MindComment',
  computed: {
    ...mapGetters(['commentData']),
    // 사용자가 이미지 설정하지 않은 경우 default 이미지 첨부
    creatorImage() {
      return this.imageError ? this.defaultImage : 'creator-image.jpg';
    },
    searchHandler() {
      return this.items.filter((elem) => {
        return elem.data.toLowerCase().includes(this.searche.toLowerCase());
      });
    },
  },
  data() {
    const no = Number(this.$route.params.no);
    return {
      no: no,
      write: '',
      searche: '',
      userId: localStorage.getItem('user-id'),
      nickName: localStorage.getItem('user-nickname'),
      // default 이미지 받아오는 경로
      defaultImage: require('@/assets/images/mindwiki_logo-color.png'),
      imageError: false,
      items: [],
      user: {
        files: '', // 프로필 사진
      },
      profile: '',
      message: '',
      members: '',
    };
  },
  methods: {
    // 댓글 생성
    createcomment() {
      const form = new FormData();
      form.append('jwt', this.$store.getters.getJWT);
      form.append('no', this.no);
      form.append('data', this.write);

      this.$store.dispatch('createComment', form).then(() => {
        this.items = this.$store.getters.commentData;
        this.readcomment();
        this.write = '';
      });
    },
    // 댓글 불러오기
    readcomment() {
      this.$store.dispatch('readComment', this.no).then(() => {
        this.items = this.$store.getters.commentData;
        this.getNickName();
      });
    },
    // 댓글 삭제
    deletecomment(commentID) {
      const form = new FormData();
      form.append('jwt', this.$store.getters.getJWT);
      form.append('no', this.no);
      form.append('commentID', commentID);
      this.$store.dispatch('deleteComment', form).then(() => {
        this.items = this.$store.getters.commentData;
        this.readcomment();
      });
    },
    // 엔터키 눌렀을 때 이벤트
    onInputKeyword: function(event) {
      this.$emit('input-change', event.target.value);
    },
    // 상세 조회 페이지로 이동
    backPage: function() {
      this.$router.push({ name: 'MindMapDetail', params: { no: Number(this.no) } });
    },
    // 닉네임 가져오기
    getNickName() {
      for (var i = 0; i < this.items.length; i++) {
        for (var j = 0; j < this.members.length; j++) {
          if (this.items[i].email === this.members[j].email) {
            if (this.members[j].nickName === null)
              this.$set(this.items[i], 'nickName', '(탈퇴회원)');
            else this.$set(this.items[i], 'nickName', this.members[j].nickName);
          }
        }
      }
    },
  },
  created: function() {
    this.readcomment();

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
        this.user.files = this.profile.profileDefaultPic;
      }
    });
  },
  mounted() {
    // 회원 목록 가져오기
    let form1 = new FormData();
    form1.append('jwt', this.$store.getters.getJWT);

    this.$store.dispatch('readMemberList', form1).then(() => {
      this.members = this.$store.getters.memberList;
      this.getNickName();
    });
  },
};
</script>

<style>
/* 상단 네비 css */
#navi_shadow {
  box-shadow: 0px 2px 4px -1px rgb(0 0 0 / 3%), 0px 4px 5px 0px rgb(0 0 0 / 3%),
    0px 1px 10px 0px rgb(0 0 0 / 3%);
  position: sticky;
  top: 0;
  z-index: 999;
}
/* 댓글 입력창 css */
#footer {
  position: fixed;
  bottom: 0;
  width: 100%;
  background: #fff;
  z-index: 999;
  height: 70px;
}
/* 이메일 css */
#commentEmail {
  max-width: 80px;
  text-align: left;
  font-size: 0.75em;
}
/* 댓글 자동 줄바꿈 */
#commentTitle {
  white-space: normal;
  line-height: 1.5rem;
  font-size: 0.85em;
  max-width: 220px;
  text-align: left;
}
/* 돋보기 위치 조정 */
#searchIcon {
  position: fixed;
  bottom: 23%;
  right: 6%;
}
/* 구분선 */
#divider {
  margin: 0;
}
/* 검색바 위치 */
.searched {
  position: fixed;
  right: 5%;
  top: 15%;
  height: 10px;
  padding: 0 !important;
  width: 50%;
}
/* 검색영역 조정 */
.v-text-field--filled > .v-input__control > .v-input__slot,
.v-text-field--full-width > .v-input__control > .v-input__slot,
.v-text-field--outlined > .v-input__control > .v-input__slot {
  align-items: stretch;
  min-height: 33px;
}
/* 돋보기 위치 조정 */
.v-text-field.v-text-field--enclosed:not(.v-text-field--rounded)
  > .v-input__control
  > .v-input__slot,
.v-text-field.v-text-field--enclosed .v-text-field__details {
  max-height: 12px;
}
</style>
<style scoped>
.v-text-field {
  padding-top: 0px !important;
  margin-top: 0px !important;
}
.v-input .v-label {
  height: auto;
  margin-top: 0.3rem;
}
@media (min-width: 320px) and (max-width: 480px) {
  .comment_list {
    max-width: 375px;
    margin: 0 auto;
  }
}

@media (min-width: 576px) {
  .comment_list {
    max-width: 540px;
    margin: 0 auto;
  }
}

@media (min-width: 768px) {
  .comment_list {
    max-width: 720px;
    margin: 0 auto;
  }
}

@media (min-width: 992px) {
  .comment_list {
    max-width: 720px;
    margin: 0 auto;
  }
}

@media (min-width: 1200px) {
  .comment_list {
    max-width: 720px;
    margin: 0 auto;
  }
}
</style>

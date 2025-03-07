<template>
  <div class="layout-header">
    <!--顶部菜单-->
    <div
      class="layout-header-left"
      v-if="navMode === 'horizontal' || (navMode === 'horizontal-mix' && mixMenu)"
    >
      <div class="logo" v-if="navMode === 'horizontal'">
        <img :src="websiteConfig.logo" alt="" />
        <h2 v-show="!collapsed" class="title">{{ websiteConfig.title }}</h2>
      </div>
      <AsideMenu
        v-model:collapsed="collapsed"
        v-model:location="getMenuLocation"
        :inverted="getInverted"
        mode="horizontal"
      />
    </div>
    <!--左侧菜单-->
    <div class="layout-header-left" v-else>
      <!-- 菜单收起 -->
      <div
        class="ml-1 layout-header-trigger layout-header-trigger-min"
        @click="() => $emit('update:collapsed', !collapsed)"
      >
        <n-icon size="18" v-if="collapsed">
          <MenuUnfoldOutlined />
        </n-icon>
        <n-icon size="18" v-else>
          <MenuFoldOutlined />
        </n-icon>
      </div>
      <!-- 刷新 -->
      <div
        class="mr-1 layout-header-trigger layout-header-trigger-min"
        v-if="headerSetting.isReload"
        @click="reloadPage"
      >
        <n-icon size="18">
          <ReloadOutlined />
        </n-icon>
      </div>
      <!-- 面包屑 -->
      <n-breadcrumb v-if="crumbsSetting.show">
        <template
          v-for="routeItem in breadcrumbList"
          :key="routeItem.name === 'Redirect' ? void 0 : routeItem.name"
        >
          <n-breadcrumb-item v-if="routeItem.meta.title">
            <n-dropdown
              v-if="routeItem.children.length"
              :options="routeItem.children"
              @select="dropdownSelect"
            >
              <span class="link-text">
                <component
                  v-if="crumbsSetting.showIcon && routeItem.meta.icon"
                  :is="routeItem.meta.icon"
                />
                {{ routeItem.meta.title }}
              </span>
            </n-dropdown>
            <span class="link-text" v-else>
              <component
                v-if="crumbsSetting.showIcon && routeItem.meta.icon"
                :is="routeItem.meta.icon"
              />
              {{ routeItem.meta.title }}
            </span>
          </n-breadcrumb-item>
        </template>
      </n-breadcrumb>
    </div>
    <div class="layout-header-right">
      <div
        class="layout-header-trigger layout-header-trigger-min"
        v-for="item in iconList"
        :key="item.icon"
      >
        <n-tooltip placement="bottom">
          <template #trigger>
            <n-icon size="18">
              <component :is="item.icon" v-on="item.eventObject || {}" />
            </n-icon>
          </template>
          <span>{{ item.tips }}</span>
        </n-tooltip>
      </div>
      <!--切换全屏-->
      <div class="layout-header-trigger layout-header-trigger-min" v-show="false">
        <n-tooltip placement="bottom">
          <template #trigger>
            <n-icon size="18">
              <component :is="fullscreenIcon" @click="toggleFullScreen" />
            </n-icon>
          </template>
          <span>全屏</span>
        </n-tooltip>
      </div>
      <div class="layout-header-trigger layout-header-trigger-min" v-show="true">
        <n-tooltip placement="bottom">
          <template #trigger>
            <n-icon size="18">
              <component :is="checkSquareOutlined" @click="checkIn" />
            </n-icon>
          </template>
          <span>签到</span>
        </n-tooltip>
      </div>
      <div class="layout-header-trigger layout-header-trigger-min" v-show="true">
        <n-tooltip placement="bottom" trigger="hover" @update:show="walletStore.syncFishCoin()">
          <template #trigger>
            <n-icon size="18">
              <component :is="walletOutlined" @click="openModal" />
            </n-icon>
          </template>
          <span>钱包 {{ walletStore.fishCoin }} 枚</span>
        </n-tooltip>
      </div>
      <!-- 个人中心 -->
      <div class="layout-header-trigger layout-header-trigger-min">
        <n-dropdown trigger="hover" @select="avatarSelect" :options="avatarOptions">
          <div class="avatar">
            <n-avatar
              round
              :src="userInfo.headerImg"
              fallback-src="https://cdn.pixabay.com/animation/2023/05/15/09/24/09-24-42-868_512.gif"
            />
          </div>
        </n-dropdown>
      </div>
      <!--设置-->
      <div
        class="layout-header-trigger layout-header-trigger-min"
        @click="openSetting"
        v-show="false"
      >
        <n-tooltip placement="bottom-end">
          <template #trigger>
            <n-icon size="18" style="font-weight: bold">
              <SettingOutlined />
            </n-icon>
          </template>
          <span>项目配置</span>
        </n-tooltip>
      </div>
    </div>
  </div>
  <!--项目配置-->
  <ProjectSetting ref="drawerSetting" />
  <!-- 钱包信息 -->
  <n-modal
    :show="walletStore.showModal"
    class="custom-card"
    preset="card"
    label-placement="center"
    :style="bodyStyle"
    title="钱包"
    size="huge"
    :bordered="false"
    :segmented="segmented"
    aria-modal="true"
    role="dialog"
    :on-after-leave="closeModal"
    :on-close="closeModal"
    :on-esc="closeModal"
    :on-mask-click="closeModal"
  >
    <template #header-extra>
      <a>鱼币: {{ walletStore.fishCoin }} 枚</a>
    </template>
    <n-space>
      <n-button secondary strong :render-icon="renderIcon" @click="checkIn">签到 +1 鱼币</n-button>
      <n-tooltip trigger="hover">
        <template #trigger>
          <n-button secondary strong :render-icon="renderTaobaoIcon" @click="bugRedeemCode"
            >购买兑换码</n-button
          >
        </template>
        【淘宝】https://m.tb.cn/h.5W9ZgaG?tk=l4fqdDsdca9 CZ0001 「开放鱼兑换码」点击链接直接打开
        或者 淘宝搜索直接打开
      </n-tooltip>
    </n-space>
    <n-divider />
    <n-space>
      <n-form
        ref="formRef"
        inline
        :label-width="80"
        :model="formValue"
        :rules="rules"
        size="medium"
      >
        <n-form-item label="兑换鱼币" path="redeemCode">
          <n-input v-model:value="formValue.redeemCode" placeholder="输入鱼币兑换码" />
        </n-form-item>
        <n-form-item>
          <n-button attr-type="button" @click="recharge">验证</n-button>
        </n-form-item>
      </n-form>
    </n-space>
    <template #footer>
      <a></a>
    </template>
  </n-modal>
  <!-- 个人中心 -->
  <n-modal
    v-model:show="showProfileModal"
    class="custom-card"
    preset="card"
    label-placement="center"
    :style="bodyStyle"
    title="个人中心"
    size="huge"
    :bordered="false"
    :segmented="segmented"
    aria-modal="true"
    role="dialog"
  >
    <div>
      <input ref="input" type="file" name="image" accept="image/*" @change="setImage" />

      <div class="content">
        <section class="cropper-area">
          <div class="img-cropper">
            <vue-cropper ref="cropper" :aspect-ratio="16 / 16" :src="imgSrc" preview=".preview" />
          </div>
          <div class="actions">
            <a href="#" role="button" @click.prevent="cropImage">裁剪</a>
            <a href="#" role="button" @click.prevent="reset"> 重置 </a>
            <a href="#" role="button" @click.prevent="showFileChooser"> 选择文件 </a>
            <a href="#" role="button" @click.prevent="submitAvatar"> 上传头像 </a>
          </div>
        </section>
        <section class="preview-area">
          <p>预览</p>
          <div class="preview"></div>
          <p>裁剪图片</p>
          <div class="cropped-image">
            <img v-if="cropImg" :src="cropImg" alt="裁剪图片" />
            <div v-else class="crop-placeholder"></div>
          </div>
        </section>
      </div>
    </div>
    <template #footer>
      <a></a>
    </template>
  </n-modal>
</template>

<script lang="ts">
  import { computed, defineComponent, h, reactive, ref, toRefs, unref } from 'vue';
  import { useRoute, useRouter } from 'vue-router';
  import components from './components';
  import { FormInst, NDialogProvider, NIcon, useDialog, useMessage } from 'naive-ui';
  import { TABS_ROUTES } from '@/store/mutation-types';
  import { UserInfoType, useUserStore } from '@/store/modules/user';
  import { useScreenLockStore } from '@/store/modules/screenLock';
  import ProjectSetting from './ProjectSetting.vue';
  import { AsideMenu } from '@/layout/components/Menu';
  import { useProjectSetting } from '@/hooks/setting/useProjectSetting';
  import { websiteConfig } from '@/config/website.config';
  import { CashOutline as CashIcon } from '@vicons/ionicons5';
  import VueCropper from 'vue-cropperjs';
  import 'cropperjs/dist/cropper.css';
  import { checkInApi, redeemFishCoin } from '@/api/transaction/transaction';
  import { setSelfInfo } from '@/api/system/user';
  import { useWalletStore } from '@/store/modules/wallet';
  import { copyToClip } from '@/utils/copy';
  import { TaobaoOutlined } from '@vicons/antd';

  export default defineComponent({
    name: 'PageHeader',
    components: {
      ...components,
      NDialogProvider,
      ProjectSetting,
      AsideMenu,
      VueCropper,
    },
    props: {
      collapsed: {
        type: Boolean,
      },
      inverted: {
        type: Boolean,
      },
    },
    emits: ['update:collapsed'],
    setup(props) {
      const userStore = useUserStore();
      const walletStore = useWalletStore();
      const useLockscreen = useScreenLockStore();
      const message = useMessage();
      const dialog = useDialog();
      const { navMode, navTheme, headerSetting, menuSetting, crumbsSetting, isMobile } =
        useProjectSetting();

      const userInfo = ref<UserInfoType>(userStore.info);
      const formRef = ref<FormInst | null>(null);
      const drawerSetting = ref();
      const imgSrc = ref(userInfo.value.headerImg || '');
      const cropImg = ref('');
      const cropper = ref();
      const input = ref();
      const cropImage = () => {
        // get image data for post processing, e.g. upload or setting image src
        cropImg.value = cropper.value.getCroppedCanvas().toDataURL();
      };
      const reset = () => {
        cropper.value.reset();
      };
      const showFileChooser = () => {
        input.value.click();
      };
      const submitAvatar = async () => {
        // 这个是裁剪的结果
        cropImg.value = cropper.value.getCroppedCanvas().toDataURL();
        if (imgSrc.value === userInfo.value.headerImg) {
          console.log('未发生改变');
          return;
        }
        // 判断上传的照片是不是gif
        const type = imgSrc.value.substring(5, 14);
        const isGIF = type === 'image/gif' || type === 'image/GIF';
        let avatar = cropImg.value;
        if (isGIF) {
          avatar = imgSrc.value;
        }
        const { code, msg } = await setSelfInfo({ headerImg: avatar });
        if (code === 0) {
          showProfileModal.value = false;
          await userStore.getInfo();
        }
        message.info(msg);
      };
      const setImage = (e) => {
        const file = e.target.files[0];

        if (file.type.indexOf('image/') === -1) {
          message.info('请上传图片文件');
          return;
        }

        if (typeof FileReader === 'function') {
          const reader = new FileReader();

          reader.onload = (event: any) => {
            imgSrc.value = event.target.result;
            // rebuild cropperjs with the updated source
            cropper.value.replace(event.target.result);
          };

          reader.readAsDataURL(file);
        } else {
          alert('Sorry, FileReader API not supported');
        }
      };
      const renderIcon = () => {
        return h(NIcon, null, {
          default: () => h(CashIcon),
        });
      };
      const renderTaobaoIcon = () => {
        return h(NIcon, null, {
          default: () => h(TaobaoOutlined),
        });
      };
      // 购买兑换码
      const bugRedeemCode = async () => {
        // 复制链接【淘宝】https://m.tb.cn/h.5W9ZgaG?tk=l4fqdDsdca9 CZ0001 「开放鱼兑换码」点击链接直接打开 或者 淘宝搜索直接打开
        const taobaoLink =
          '【淘宝】https://m.tb.cn/h.5W9ZgaG?tk=l4fqdDsdca9 CZ0001 「开放鱼兑换码」\n' +
          '点击链接直接打开 或者 淘宝搜索直接打开';
        const taobaoUrl = 'https://m.tb.cn/h.5W9ZgaG?tk=l4fqdDsdca9';
        await copyToClip(taobaoLink);
        window.open(taobaoUrl, '_blank');
      };
      // 签到
      const checkIn = async () => {
        const { msg } = await checkInApi();
        await walletStore.syncFishCoin();
        message.info(msg);
      };
      const showProfileModal = ref(false);
      const formValue = ref({
        redeemCode: '',
      });
      const rules = {
        redeemCode: {
          required: true,
          message: '请输入鱼币兑换码',
          trigger: ['input'],
        },
      };
      const state = reactive({
        fullscreenIcon: 'FullscreenOutlined',
        walletOutlined: 'WalletOutlined',
        checkSquareOutlined: 'CheckSquareOutlined',
        navMode,
        navTheme,
        headerSetting,
        crumbsSetting,
      });

      const getInverted = computed(() => {
        return ['light', 'header-dark'].includes(unref(navTheme))
          ? props.inverted
          : !props.inverted;
      });

      const bodyStyle = {
        width: '400px',
      };

      const segmented = {
        content: 'soft',
        footer: 'soft',
      } as const;

      const mixMenu = computed(() => {
        return unref(menuSetting).mixMenu;
      });

      const getChangeStyle = computed(() => {
        const { collapsed } = props;
        const { minMenuWidth, menuWidth } = unref(menuSetting);
        return {
          left: collapsed ? `${minMenuWidth}px` : `${menuWidth}px`,
          width: `calc(100% - ${collapsed ? `${minMenuWidth}px` : `${menuWidth}px`})`,
        };
      });

      const getMenuLocation = computed(() => {
        return 'header';
      });

      const router = useRouter();
      const route = useRoute();

      const generator: any = (routerMap) => {
        return routerMap.map((item) => {
          const currentMenu = {
            ...item,
            label: item.meta.title,
            key: item.name,
            disabled: item.path === '/',
          };
          // 是否有子菜单，并递归处理
          if (item.children && item.children.length > 0) {
            // Recursion
            currentMenu.children = generator(item.children, currentMenu);
          }
          return currentMenu;
        });
      };

      const breadcrumbList = computed(() => {
        return generator(route.matched);
      });

      const dropdownSelect = (key) => {
        router.push({ name: key });
      };

      // 刷新页面
      const reloadPage = () => {
        router.push({
          path: '/redirect' + unref(route).fullPath,
        });
      };

      // 退出登录
      const doLogout = () => {
        dialog.info({
          title: '提示',
          content: '您确定要退出登录吗',
          positiveText: '确定',
          negativeText: '取消',
          onPositiveClick: () => {
            userStore.logout().then(() => {
              message.success('成功退出登录');
              // 移除标签页
              localStorage.removeItem(TABS_ROUTES);
              router
                .replace({
                  name: 'Login',
                  query: {
                    redirect: route.fullPath,
                  },
                })
                .finally(() => location.reload());
            });
          },
          onNegativeClick: () => {},
        });
      };

      // 切换全屏图标
      const toggleFullscreenIcon = () =>
        (state.fullscreenIcon =
          document.fullscreenElement !== null ? 'FullscreenExitOutlined' : 'FullscreenOutlined');

      // 监听全屏切换事件
      document.addEventListener('fullscreenchange', toggleFullscreenIcon);

      // 全屏切换
      const toggleFullScreen = () => {
        if (!document.fullscreenElement) {
          document.documentElement.requestFullscreen();
        } else {
          if (document.exitFullscreen) {
            document.exitFullscreen();
          }
        }
      };

      const openModal = () => {
        walletStore.setShowModal(true);
        walletStore.syncFishCoin();
        formValue.value.redeemCode = '';
      };
      const closeModal = () => {
        walletStore.setShowModal(false);
        formValue.value.redeemCode = '';
      };
      const recharge = async (e: MouseEvent) => {
        e.preventDefault();
        formRef.value?.validate(async (errors) => {
          if (errors) {
            message.error('请输入鱼币兑换码');
            return;
          }
          // 向后端发起验证
          const { msg } = await redeemFishCoin({
            redeemCode: formValue.value.redeemCode,
          });
          await walletStore.syncFishCoin();
          message.info(msg);
        });
      };

      // 图标列表
      const iconList = [
        // {
        //   icon: 'SearchOutlined',
        //   tips: '搜索',
        // },
        {
          icon: 'GithubOutlined',
          tips: 'github',
          eventObject: {
            click: () => window.open('https://github.com/oldweipro'),
          },
        },
        {
          icon: 'LockOutlined',
          tips: '锁屏',
          eventObject: {
            click: () => useLockscreen.setLock(true),
          },
        },
      ];
      const avatarOptions = [
        {
          label: '个人设置',
          key: 1,
        },
        {
          label: '退出登录',
          key: 2,
        },
      ];

      //头像下拉菜单
      const avatarSelect = (key) => {
        switch (key) {
          case 1:
            showProfileModal.value = true;
            break;
          case 2:
            doLogout();
            break;
        }
      };

      function openSetting() {
        const { openDrawer } = drawerSetting.value;
        openDrawer();
      }

      return {
        ...toRefs(state),
        isMobile,
        iconList,
        toggleFullScreen,
        recharge,
        openModal,
        closeModal,
        doLogout,
        route,
        dropdownSelect,
        avatarOptions,
        getChangeStyle,
        avatarSelect,
        breadcrumbList,
        reloadPage,
        drawerSetting,
        openSetting,
        getInverted,
        getMenuLocation,
        mixMenu,
        websiteConfig,
        showProfileModal,
        bodyStyle,
        segmented,
        formRef,
        formValue,
        rules,
        imgSrc,
        cropImg,
        setImage,
        cropImage,
        reset,
        showFileChooser,
        submitAvatar,
        input,
        cropper,
        userInfo,
        checkIn,
        bugRedeemCode,
        renderIcon,
        renderTaobaoIcon,
        walletStore,
      };
    },
  });
</script>

<style lang="less" scoped>
  .layout-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0;
    height: 52px;
    box-shadow: 0 1px 4px rgb(0 21 41 / 8%);
    transition: all 0.2s ease-in-out;
    width: 100%;
    z-index: 11;

    &-left {
      display: flex;
      align-items: center;

      .logo {
        display: flex;
        align-items: center;
        justify-content: center;
        height: 64px;
        line-height: 64px;
        overflow: hidden;
        white-space: nowrap;
        padding-left: 10px;

        img {
          width: auto;
          height: 32px;
          margin-right: 10px;
        }

        .title {
          margin-bottom: 0;
        }
      }

      ::v-deep(.ant-breadcrumb span:last-child .link-text) {
        color: #515a6e;
      }

      .n-breadcrumb {
        display: inline-block;
      }

      &-menu {
        color: var(--text-color);
      }
    }

    &-right {
      display: flex;
      align-items: center;
      margin-right: 20px;

      .avatar {
        display: flex;
        align-items: center;
        height: 64px;
      }

      > * {
        cursor: pointer;
      }
    }

    &-trigger {
      display: inline-block;
      width: 64px;
      height: 64px;
      text-align: center;
      cursor: pointer;
      transition: all 0.2s ease-in-out;

      .n-icon {
        display: flex;
        align-items: center;
        height: 64px;
        line-height: 64px;
      }

      &:hover {
        background: hsla(0, 0%, 100%, 0.08);
      }

      .anticon {
        font-size: 16px;
        color: #515a6e;
      }
    }

    &-trigger-min {
      width: auto;
      padding: 0 12px;
    }
  }

  .layout-header-light {
    background: #fff;
    color: #515a6e;

    .n-icon {
      color: #515a6e;
    }

    .layout-header-left {
      ::v-deep(.n-breadcrumb .n-breadcrumb-item:last-child .n-breadcrumb-item__link) {
        color: #515a6e;
      }
    }

    .layout-header-trigger {
      &:hover {
        background: #f8f8f9;
      }
    }
  }

  .layout-header-fix {
    position: fixed;
    top: 0;
    right: 0;
    left: 200px;
    z-index: 11;
  }

  //::v-deep(.menu-router-link) {
  //  color: #515a6e;
  //
  //  &:hover {
  //    color: #1890ff;
  //  }
  //}

  input[type='file'] {
    display: none;
  }

  .content {
    display: flex;
  }

  .cropper-area {
    width: 214px;
    margin-right: 10px;
  }

  .actions {
    margin-top: 1rem;
  }

  .actions a {
    display: inline-block;
    padding: 5px 15px;
    background: #0062cc;
    color: white;
    text-decoration: none;
    border-radius: 3px;
    margin-right: 1rem;
    margin-bottom: 1rem;
  }

  .preview-area {
    width: 67px;
  }

  .preview-area p {
    font-size: 1.25rem;
    margin: 0 0 1rem;
  }

  .preview-area p:last-of-type {
    margin-top: 1rem;
  }

  .preview {
    width: 100%;
    height: calc(67px * (16 / 16));
    overflow: hidden;
  }

  .crop-placeholder {
    width: 67px;
    height: 67px;
    background: #ccc;
  }

  .cropped-image img {
    max-width: 100%;
  }
</style>

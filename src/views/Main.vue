<style lang="less">
    @import "./main.less";
</style>
<template>
    <div class="main" :class="{'main-hide-text': shrink}">
        <div class="sidebar-menu-con" :style="{width: shrink?'60px':'200px', overflow: shrink ? 'visible' : 'auto'}">
            <shrinkable-menu
                    :shrink="shrink"
                    @on-change="handleSubmenuChange"
                    :theme="menuTheme"
                    :before-push="beforePush"
                    :open-names="openedSubmenuArr"
                    :menu-list="menuList">
                <div slot="top" class="logo-con" style="cursor: pointer">
                    <img  @click="toHome" v-show="!shrink" style="margin-top: 20px;margin-bottom: 10px" src="../images/tokenup1.png" key="max-logo"/>
                    <img  @click="toHome" v-show="shrink" src="../images/TOUP_min.png" key="min-logo"/>
                </div>
            </shrinkable-menu>
        </div>
        <div class="main-header-con" :style="{paddingLeft: shrink?'60px':'200px'}">
            <div class="main-header">
                <div class="navicon-con">
                    <Button :style="{transform: 'rotateZ(' + (this.shrink ? '-90' : '0') + 'deg)'}" type="text"
                                     @click="toggleClick">
                    <Icon type="navicon" size="32"></Icon>
                </Button>
                </div>
                <div class="header-middle-con">
                    <div class="main-breadcrumb">
                        <breadcrumb-nav :currentPath="currentPath"></breadcrumb-nav>
                    </div>
                </div>
                <div class="header-avator-con">
                    <full-screen v-model="isFullScreen" @on-change="fullscreenChange"></full-screen>
                    <lock-screen></lock-screen>
                    <theme-switch></theme-switch>

                    <div class="user-dropdown-menu-con">
                        <Row type="flex" justify="end" align="middle" class="user-dropdown-innercon">
                            <Dropdown transfer trigger="click" @on-click="handleClickUserDropdown">
                                <a href="javascript:void(0)">
                                    <span class="main-user-name">{{ username }}</span>
                                    <Icon type="arrow-down-b"></Icon>
                                </a>
                                <DropdownMenu slot="list">
                                    <DropdownItem name="changePassword">修改密码</DropdownItem>
                                    <DropdownItem name="loginout" divided>退出登录</DropdownItem>
                                </DropdownMenu>
                            </Dropdown>
                            <Avatar :src="avatorPath" style="background: #619fe7;margin-left: 10px;"></Avatar>
                        </Row>
                    </div>
                </div>
            </div>
            <div class="tags-con" id="parentMenu">
                <tags-page-opened :pageTagsList="pageTagsList"></tags-page-opened>
            </div>
        </div>
        <div class="single-page-con" :style="{left: shrink?'60px':'200px'}">
            <div class="single-page">
                <keep-alive :include="cachePage">
                    <router-view></router-view>
                </keep-alive>
            </div>
        </div>
        <Modal v-model="editPasswordModal" :closable='false' :mask-closable=false :width="500">
            <h3 slot="header" style="color:#2D8CF0">修改密码</h3>
            <Form ref="editPasswordForm" :model="editPasswordForm" :label-width="100" label-position="right" :rules="passwordValidate">
                <FormItem label="原密码" prop="oldPass" :error="oldPassError">
                    <Input v-model="editPasswordForm.oldPass" placeholder="请输入现在使用的密码" ></Input>
                </FormItem>
                <FormItem label="新密码" prop="newPass">
                    <Input v-model="editPasswordForm.newPass" placeholder="请输入新密码，至少6位字符" ></Input>
                </FormItem>
                <FormItem label="确认新密码" prop="rePass">
                    <Input v-model="editPasswordForm.rePass" placeholder="请再次输入新密码" ></Input>
                </FormItem>
            </Form>
            <div slot="footer">
                <Button type="text" @click="cancelEditPass">取消</Button>
                <Button type="primary" :loading="savePassLoading" @click="saveEditPass">保存</Button>
            </div>
        </Modal>
    </div>
</template>
<script>
    import shrinkableMenu from './main-components/shrinkable-menu/shrinkable-menu.vue';
    import tagsPageOpened from './main-components/tags-page-opened.vue';
    import breadcrumbNav from './main-components/breadcrumb-nav.vue';
    import fullScreen from './main-components/fullscreen.vue';
    import lockScreen from './main-components/lockscreen/lockscreen.vue';
    import themeSwitch from './main-components/theme-switch/theme-switch.vue';
    import util from '@/libs/util.js';
    import API from '@/api';

    export default {
        components: {
            shrinkableMenu,
            tagsPageOpened,
            breadcrumbNav,
            fullScreen,
            lockScreen,
            themeSwitch
        },
        data() {
            const valideRePassword = (rule, value, callback) => {
                if (value !== this.editPasswordForm.newPass) {
                    callback(new Error('两次输入密码不一致'));
                } else {
                    callback();
                }
            };
            return {
                shrink: false,
                username: '',
                isFullScreen: false,
                openedSubmenuArr: this.$store.state.app.openedSubmenuArr,
                editPasswordModal: false, // 修改密码模态框显示
                savePassLoading: false,
                oldPassError: '',
                editPasswordForm: {
                    oldPass: '',
                    newPass: '',
                    rePass: ''
                },
                passwordValidate: {
                    oldPass: [
                        { required: true, message: '请输入原密码', trigger: 'blur' }
                    ],
                    newPass: [
                        { required: true, message: '请输入新密码', trigger: 'blur' },
                        { min: 6, message: '请至少输入6个字符', trigger: 'blur' },
                        { max: 32, message: '最多输入32个字符', trigger: 'blur' }
                    ],
                    rePass: [
                        { required: true, message: '请再次输入新密码', trigger: 'blur' },
                        { validator: valideRePassword, trigger: 'blur' }
                    ]
                },
            };
        },
        computed: {
            menuList() {
                return localStorage.menuList ? JSON.parse(localStorage.menuList) : [];
            },
            pageTagsList() {
                return this.$store.state.app.pageOpenedList; // 打开的页面的页面对象
            },
            currentPath() {
                return this.$store.state.app.currentPath; // 当前面包屑数组
            },
            avatorPath() {
                return localStorage.avatorImgPath;
            },
            cachePage() {
                return this.$store.state.app.cachePage;
            },
            lang() {
                return this.$store.state.app.lang;
            },
            menuTheme() {
                return this.$store.state.app.menuTheme;
            }
        },
        methods: {
            init() {
                this.username = this.$cookie.get('username');
                let pathArr = util.setCurrentPath(this, this.$route.name);
                this.$store.commit('updateMenulist');
                if (pathArr.length >= 2) {
                    this.$store.commit('addOpenSubmenu', pathArr[1].name);
                }
                this.checkTag(this.$route.name);
                this.$store.commit('setMessageCount', 3);
            },
            toHome(){
                this.$router.push({
                    name: 'home_index'
                });
            },
            toggleClick() {
                this.shrink = !this.shrink;
            },
            handleClickUserDropdown(name) {
                if (name === 'changePassword') {
                    this.editPasswordModal = true;
                } else if (name === 'loginout') {
                    API.common.logout().then(({data}) => {
                        if (data && data.errCode === 0) {
                            // 退出登录
                            this.$store.commit('logout', this);
                            this.$store.commit('clearOpenedSubmenu');
                            this.$router.replace({name: 'login'});
                        }else{
                            this.$Message.error(data.errMsg);
                        }
                    }).catch(()=>{
                        this.$Message.error('连接失败，请检查网络！');
                    });
                }
            },
            checkTag(name) {
                let openpageHasTag = this.pageTagsList.some(item => {
                    if (item.name === name) {
                        return true;
                    }
                });
                if (!openpageHasTag) { //  解决关闭当前标签后再点击回退按钮会退到当前页时没有标签的问题
                    util.openNewPage(this, name, this.$route.params || {}, this.$route.query || {});
                }
            },
            handleSubmenuChange(val) {
            },
            beforePush() {
                return true;
            },
            fullscreenChange(isFullScreen) {
            },
            cancelEditPass () {
                this.editPasswordModal = false;
            },
            saveEditPass () {
                this.$refs.editPasswordForm.validate((valid) => {
                    if (valid) {
                        let passwordParams = {
                            'username':this.$cookie.get('username'),
                            'password': this.editPasswordForm.oldPass,
                            'newPassword': this.editPasswordForm.newPass
                        }
                        this.savePassLoading = true;
                        API.manager.updatePassword(passwordParams).then(({data}) => {
                            this.savePassLoading = false;
                            if (data && data.errCode === 0) {
                                this.$Message.success("修改成功");
                                this.editPasswordModal = false;
                            }else{
                                this.$Message.error(data.errMsg);
                            }
                        }).catch(()=>{
                            this.savePassLoading = false;
                            this.$Message.error('连接失败，请检查网络！');
                        });
                    } else {
                        this.$Message.error('请填写合适的信息！');
                    }
                })

            },
        },
        watch: {
            '$route'(to) {
                this.$store.commit('setCurrentPageName', to.name);
                let pathArr = util.setCurrentPath(this, to.name);
                if (pathArr.length > 2) {
                    this.$store.commit('addOpenSubmenu', pathArr[1].name);
                }
                this.checkTag(to.name);
                localStorage.currentPageName = to.name;
            },
            lang() {
                util.setCurrentPath(this, this.$route.name); // 在切换语言时用于刷新面包屑
            }
        },
        mounted() {
            this.init();
        },
        created() {
            // 显示打开的页面的列表
            this.$store.commit('setOpenedList');
        }
    };
</script>

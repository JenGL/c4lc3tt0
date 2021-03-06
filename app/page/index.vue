<template>
    <div class="page-container">
        <md-app md-mode="reveal">
            <md-app-toolbar class="md-primary">
                <Header :hideLogInfo=false :logged=logged v-on:menu-click="toggleMenu"></Header>
            </md-app-toolbar>
            <md-app-drawer :md-active.sync="menuVisible">
                <Menu :logged=logged :user=user menuVisible=menuVisible v-on:menu-logout="logOut"
                      v-on:click-league="switchLeague($event)"></Menu>
            </md-app-drawer>

            <md-app-content>
                <Matches v-if="content.matches" v-on:click-match="showTab('match', $event)"></Matches>
                <Players v-if="content.players" v-on:click-player="showTab('player', $event)"></Players>
                <Teams v-if="content.teams"></Teams>
                <Player v-if="content.player" v-on:click-match="showTab('match', $event)"></Player>
                <Match v-if="content.match" v-on:click-player="showTab('player', $event)"></Match>
                <AddPlayer v-if="content.addplayer"></AddPlayer>
                <AddMatch v-if="content.addmatch"></AddMatch>
            </md-app-content>
        </md-app>
        <md-speed-dial v-if="admin" class="md-bottom-right" md-direction="top">
            <md-speed-dial-target>
                <md-icon>add</md-icon>
            </md-speed-dial-target>

            <md-speed-dial-content>
                <md-button class="md-icon-button" @click="showTab('addplayer')">
                    <md-icon>people</md-icon>
                </md-button>
                <md-button class="md-icon-button" @click="showTab('addmatch')">
                    <md-icon>event</md-icon>
                </md-button>
            </md-speed-dial-content>
        </md-speed-dial>
    </div>
</template>

<script>
    import Header from './components/header.vue';
    import Menu from './components/menu.vue';
    import Matches from './components/matches.vue';
    import Players from './components/players.vue';
    import Teams from './components/teams.vue';
    import Player from './components/player.vue';
    import Match from './components/match.vue';
    import API from '../js/api';
    import User from "../js/User";
    import TokenManager from "../js/token_manager";
    import AddPlayer from './components/addplayer.vue';
    import AddMatch from './components/addmatch.vue';

    export default {
        name: 'Home',
        components: {
            Header,
            Menu,
            Players,
            Teams,
            Matches,
            Player,
            Match,
            AddPlayer,
            AddMatch
        },
        props: ['tab', 'id'],
        data: () => ({
            menuVisible: false,
            logged: false,
            admin: false,
            selectedPlayer: null,
            selectedMatch: null,
            content: {
                players: false,
                matches: false,
                teams: false,
                player: false,
                match: false,
                addplayer: false,
                addmatch: false
            },
            user: {
                name: "",
                league: {},
                leagues: []
            }
        }),
        created: function () {
            API.onError((err) => {
                if (err.error === "Not authorized") this.logOut()
            });
        },
        mounted() {
            const success = () => {
                this.logged = true;
                this.user.name = User.getInstance().name;
                this.user.league = User.getInstance().currentLeague;
                this.user.leagues = User.getInstance().leagues;
                if (User.getInstance().currentLeague.admin) {
                    this.admin = true;
                }
            };

            const fail = () => {
                this.logged = false;
                this.user.name = "";
                this.user.league = {};
                this.user.leagues = [];
                this.admin = false;
                TokenManager.deleteToken();
            };

            this.content[this.tab] = true;
            if (TokenManager.isTokenValid() && User.created) {
                success();
            } else if (TokenManager.isTokenValid() && !User.created) {
                API.loginByToken()
                    .then(() => success())
                    .catch(() => fail());
            } else {
                fail();
            }
        },
        methods: {
            showTab(tab, id) {
                const path = id ? "/" + tab + "/" + id : "/" + tab;
                this.$router.push(path);
            },
            logOut() {
                TokenManager.deleteToken();
                User.getInstance().destroy();
                this.logged = false;
                this.toggleMenu(false);
            },
            toggleMenu(visible) {
                if (visible != null)
                    this.menuVisible = visible;
                else
                    this.menuVisible = !this.menuVisible;
            },
            switchLeague(league) {
                User.getInstance().switchLeague(league);
                this.$router.push("/");
            }
        }
    }
</script>

<style lang="scss" scoped>
    .page-container {
        height: 100%;
    }

    .md-app {
        height: 100%;
    }

    // Demo purposes only
    .md-drawer {
        width: 230px;
        max-width: calc(100vw - 125px);
    }

    .float-right {
        float: right;
    }

    .margin-5 {
        margin: 0 5px;
    }

    .color-white {
        color: white;
    }
</style>
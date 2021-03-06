<template>
	<header :style="[ isStandalone ? { 'top' : '0 !important' } : '']">
		<a href="#" alt="Back Button" v-if="showBackButton" v-bind:class="'header__back-button'" @click.prevent="goBack"></a>
        <la-dropdown
            @click="setLanguage"
            :options="dropdownOptions"
            :button-text="currentLanguage"
            :inner-text="$t('app.select-language')" />	
        <h4 v-if="showBackButton">{{pageName}}</h4>
        <nav v-else>
            <router-link v-for="menu in ['books', 'authors']" :key="menu" :to="{ name: menu }" :class="{ 'current' : $t(`${menu}.${menu}`) == pageName }">
                {{$t(`${menu}.${menu}`)}}
            </router-link>
        </nav>
	</header>
</template>

<script>
import { EventBus, Events } from '@/utils/eventBus.js';
import LaDropdown from '@/components/la-dropdown';
import loadjs from "loadjs";

export default {
    props:{
        backButtonRoute: {
            default: null,
            required: false,
        },
        pageName:{
            default:"",
            required:false
        },
        showLanguagePicker:{
            default:true,
            required:false
        },
        showBackButton: {
            default: true,
            required: false
        }
    },
    data() {
        return {
            showLanguageSelection: false,
            availableLanguages: [
                { code: 'af', name: 'Afrikaans'},
                { code: 'bg', name: 'Bulgarian'},
                { code: 'de', name: 'Deutsch'},
                { code: 'dk', name: 'Dansk'},
                { code: 'es', name: 'Español'},
                { code: 'en', name: 'English'},
                { code: 'fr', name: 'Français'},
                { code: 'it', name: 'Italiano'},
                { code: 'hu', name: 'Magyar'},
                { code: 'nl', name: 'Nederlands'},
                { code: 'no', name: 'Norsk' },
                { code: 'pl', name: 'Polski'},
                { code: 'pt', name: 'Português'},
                { code: 'ro', name: 'Româneste'},
                { code: 'ru', name: 'Russian'},
                { code: 'fi', name: 'Suomi'},
                { code: 'tr', name: 'Türkçe'},
            ],
            appLanguages:['no', 'en', 'de', 'fr', 'nl', 'fi']
        }
    },
    methods: {
        setLanguage(lang){
            this.showLanguageSelection = false;
            this.$store.commit('session/setAppLanguage', lang);
            this.$i18n.locale = lang;
            EventBus.$emit(Events.CONTENT_LANGUAGE_CHANGED);
        },
        goBack() {
            if(!this.backButtonRoute) {
                window.history.back();
                return; 
            }
            this.$router.push(this.backButtonRoute)  
        },
        initTopbar() {
            var scriptId = "script-bcc-topbar";
            var self = this;
            if (document.getElementById(scriptId) == null) {
                var scriptPath = "https://widgets.bcc.no/widgets/TopbarJs";
                loadjs([scriptPath,"https://widgets.bcc.no/styles/widgets.css"], {
                    async: true,
                    success: function () {
                        self.$store.commit('session/setTopbarInitialized', true);
                    },
                    error: function (path) {
                        console.error('could not load resource: ' + path)
                    },
                    before: function (path, element) {
                        if(path === scriptPath ){
                            element.id = scriptId;
                            element.setAttribute("data-authentication-type", "SPA");
                            element.setAttribute(
                                "data-authentication-location",
                                "oidc.user:https://login.bcc.no:X0ac7C8sROIhEzRGLJPFpLCZAlKGK4KV.access_token"
                            );
                        }
                    }
                });
            }
        },
    },
    mounted: function(){
        if (!this.isStandalone)
            this.initTopbar()
        else
            this.$store.commit('session/setTopbarInitialized', true);
    },
    computed: {
        isStandalone() {
            return window.matchMedia('(display-mode: standalone)').matches
        },
        currentLanguage() {
            const code = this.$store.state.session.appLanguage;
            const lang = this.availableLanguages.find((l) => l.code === code);
            return lang ? lang.name : 'Not set';
        },
        dropdownOptions() {
            return this.availableLanguages.map((l) => {
                return { value: l.code, title: l.name }
            })
        }
    },
    components:{
        LaDropdown
    }
}
</script>
<style scoped>
@media screen and (max-width: 500px){
    header .dropdown, header .filter-search {
        margin: 8px 5px;
    }
    nav, header h4 {
        -webkit-transform: none;
        transform: none;
        float: right;
        position: relative;
        width: 50%;
        left: auto;
    }
}

@media screen and (min-width: 501px) and (max-width: 648px){
    header h4 {
        width: 50%;
        left: 25%;
    }
}
</style>
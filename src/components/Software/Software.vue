<template>
    <div class="software">
        <tabs v-model="currentTabIndex">
            <tab title="Software list">
                <p class="tab-content">
                    <software-list @addSoftwareTab="goToSoftwareTab" :is-admin="isAdmin"
                                   v-bind:refresh.sync="refreshDatatable"></software-list>
                </p>
            </tab>
            <tab v-for="software in softwareTabs" :key="software.id" :title="software.fullName">
                <button class="btn pull-right" @click="removeSoftwareTab(software)">Close</button>
                <software-detail :software="software" :softwareUserRepositories="softwareUserRepositories" :is-admin="isAdmin"
                                 :parameterConstraints="parameterConstraints" @delete-software="deleteSoftware"
                                 @update-software="updateSoftware"></software-detail>
            </tab>
        </tabs>
    </div>

</template>

<script>
    import {Tabs, Tab} from 'uiv'
    import SoftwareList from "./SoftwareList";
    import SoftwareDetail from "./SoftwareDetail"

    export default {
        name: 'Software',
        components: {
            SoftwareList,
            SoftwareDetail,
            Tabs,
            Tab
        },
        data() {
            return {
                softwareTabs: [],
                maxTabs: 6,
                currentTabIndex: 0,
                refreshDatatable: false,

                isAdmin: false,

                // TODO: externalize in VueX.
                softwareUserRepositories: [],
                parameterConstraints: [],
            }
        },
        watch: {
            currentTabIndex(newValue) {
                console.log(newValue);
                if (newValue == 0) {
                    history.pushState({}, "Cytomine", "#software");
                    window.app.status.currentSoftwareId = null
                }
                else {
                    let idSoftware = this.softwareTabs[newValue-1].id;
                    console.log("push" + idSoftware);
                    history.pushState({}, "Cytomine", "#software-" + idSoftware);
                    window.app.status.currentSoftwareId = idSoftware
                }

            }
        },
        methods: {
            findSoftwareIndex(software) {
                return this.softwareTabs.findIndex(s => {
                    return s.id == software.id;
                });
            },
            goToSoftwareTab(software) {
                let index = this.findSoftwareIndex(software);
                if (index != -1)
                    this.currentTabIndex = index + 1;
                else
                    this.addSoftwareTab(software);
            },
            addSoftwareTab(software) {
                if (software && !software.name) {
                    api.get(`api/software/${software.id}.json`).then(response => {
                        software = response.data;
                        this.softwareTabs.push(software);
                        this.softwareTabs.splice(0, this.softwareTabs.length - this.maxTabs);
                        this.currentTabIndex = this.softwareTabs.length;
                    })
                } else {
                    this.softwareTabs.push(software);
                    this.softwareTabs.splice(0, this.softwareTabs.length - this.maxTabs);
                    this.currentTabIndex = this.softwareTabs.length;
                }

            },
            removeSoftwareTab(software) {
                let index = this.findSoftwareIndex(software);
                if (index != -1) {
                    this.softwareTabs.splice(index, 1);
                    this.currentTabIndex = this.softwareTabs.length
                }
            },
            deleteSoftware(software) {
                this.removeSoftwareTab(software);
                this.currentTabIndex = 0;
                this.refreshDatatable = true;
            },
            updateSoftware(software) {
                let index = this.findSoftwareIndex(software);
                this.$set(this.softwareTabs, index, software);
                this.refreshDatatable = true;
            },
            checkRoute() {
                // DEPENDS ON [BACKBONE]
                this.currentRoute = Backbone.history.getFragment();
            },
            checkAdmin() {
                // DEPENDS ON [BACKBONE]
                this.isAdmin = window.app.status.user.model.get("adminByNow");
            },
            checkCurrentSoftwareId() {
                let idSoftware = window.app.status.currentSoftwareId;
                console.log("cvjedsc" + idSoftware)
                if (!idSoftware)
                    this.currentTabIndex = 0;
                else
                    this.goToSoftwareTab({id: idSoftware})
            }
        },
        created() {
            this.checkCurrentSoftwareId();
            // TODO: externalize in VueX
            api.get(`api/software_user_repository.json`).then(response => {
                this.softwareUserRepositories = response.data.collection;
            });

            api.get(`api/parameter_constraint.json`).then(response => {
                this.parameterConstraints = response.data.collection;
            });

            // DEPENDS ON [BACKBONE]
            setInterval(this.checkRoute.bind(this), 1000);
            setInterval(this.checkAdmin.bind(this), 1000);
            setInterval(this.checkCurrentSoftwareId.bind(this), 1000);
        },
    }
</script>

<style>
    .software .tab-content {
        padding: 1em;
        padding-top: 40px;
    }

    .software .nav-tabs {
        position: fixed;
        z-index: 1000;
        width: 100%;
        background-color: white;
    }
</style>

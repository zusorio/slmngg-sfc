<template>
    <div class="container">
        <h1 class="big mb-3">Teams</h1>
        <input type="text" class="form-control mb-3" placeholder="Start typing to filter" v-model="search">
        <h1><LoadingIcon v-if="!search && !sortedEvents.length"></LoadingIcon></h1>
        <EventTeamsDisplay class="mb-4" v-for="eventID in sortedEvents" :key="eventID" :event-i-d="eventID"></EventTeamsDisplay>
        <b-pagination @page-click="scrollToTop()" v-if="!searchEvents" v-model="page" :per-page="eventsPerPage" :total-rows="events.length" align="center"></b-pagination>
    </div>
</template>

<script>
import { ReactiveRoot } from "@/utils/reactive";
import { searchInCollection } from "@/utils/search";
import LoadingIcon from "@/components/website/LoadingIcon";
import { BPagination } from "bootstrap-vue";
import EventTeamsDisplay from "@/views/lists/EventTeamsDisplay.vue";

export default {
    name: "Teams",
    components: {
        EventTeamsDisplay,
        LoadingIcon,
        BPagination
    },
    data: function() {
        return {
            search: null,
            page: 1,
            eventsPerPage: 10
        };
    },
    computed: {
        events() {
            const events = ReactiveRoot("special:public-events")?.events;
            if (!events) return [];
            return events.reverse();
        },
        searchEvents() {
            return (this.search && this.search.length > 2);
        },
        sortedEvents() {
            if (!this.searchEvents) {
                // group and paginate
                return this.events.slice(this.eventsPerPage * (this.page - 1), this.eventsPerPage * this.page);
            }
            return this.events
                .map(e => {
                    if (this.search && this.search.length > 2 && e.teams) {
                        return {
                            ...e,
                            teams: searchInCollection(e.teams, this.search, "name")
                        };
                    }
                    return e;
                }).filter(e => e.show_in_events && e.teams && e.teams.length !== 0);
        }
    },
    metaInfo() {
        return {
            title: "Teams"
        };
    },
    methods: {
        scrollToTop() {
            window.scrollTo({
                top: 0,
                behavior: "instant"
            });
        }
    }
};
</script>

<style scoped>
.pagination {
    user-select: none;
}
</style>

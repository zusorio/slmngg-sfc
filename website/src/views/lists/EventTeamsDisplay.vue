<template>
    <div class="event">
        <EventDisplay class="team-display" :event="event"/>
        <div class="event-teams d-flex row">
            <div class="team col-6 col-sm-6 col-md-4 col-lg-3 col-xl-2 tight-col" v-for="team in event.teams" :key="team.id">
                <TeamDisplay :team="team" />
            </div>
        </div>
    </div>
</template>

<script>
import { ReactiveArray, ReactiveRoot, ReactiveThing } from "@/utils/reactive";
import EventDisplay from "@/views/lists/EventDisplay.vue";
import TeamDisplay from "@/views/lists/TeamDisplay.vue";

export default {
    name: "EventTeamsDisplay",
    components: { EventDisplay, TeamDisplay },
    props: {
        eventID: String
    },
    computed: {
        event() {
            return ReactiveRoot(this.eventID, {
                theme: ReactiveThing("theme"),
                teams: ReactiveArray("teams", {
                    theme: ReactiveThing("theme")
                })
            });
        }
    }
};
</script>

<style scoped>
.tight-col {
    --pad: 6px;
    padding-left: var(--pad);
    padding-right: var(--pad);
    margin: var(--pad) 0;
}
.event-teams.row {
    margin: 0 -6px;
}
</style>

<template>
    <div class="tally-block" :class="{ preview: state === 'preview', active: state === 'active' }"
         @click="showProducerInfo = !showProducerInfo">
        <div class="d-flex flex-column align-items-center">
            <div class="state">
                {{ state.toLocaleUpperCase() }}
            </div>
            <div class="metadata d-flex flex-column align-items-center">
                <div>
                    <span v-if="number || selfObserverNumber">{{ number || selfObserverNumber }} </span>
                    <span v-else><i class="fas fa-exclamation-circle"></i> Not assigned </span>
                    <span>&middot; {{ client.name }}</span>
                </div>
            </div>
        </div>
        <div class="tally-overlay">
            <div class="team-info" v-if="teams">
                <div class="team" v-for="(team, i) in teams" :key="team.id" :style="{order: (2 * i) + 1}">
                    <div class="team-name">{{ team.name }}</div>
                    <ThemeLogo class="team-logo" :theme="team.theme" border-width=".3em" icon-padding="1em"  />
                    <div class="team-score">{{ scores[i] }}</div>
                </div>
                <div class="first-to">{{ liveMatch.first_to ? `FT${liveMatch.first_to}` : 'vs' }}</div>
            </div>
            <div class="spacer h-100"></div>
            <div class="prod-info" v-if="producer">
                <div class="prod-name flex-center">Producer: {{ producer.name }}</div>
                <div class="prod-scenes">
                    <div class="prod-preview">{{ producerPreviewScene }}</div>
                    <div class="prod-program">{{ producerProgramScene }}</div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import { ReactiveArray, ReactiveRoot, ReactiveThing } from "@/utils/reactive";
import ThemeLogo from "@/components/website/ThemeLogo.vue";

export default {
    name: "TallyViewer",
    components: { ThemeLogo },
    props: ["client", "scene"],
    sockets: {
        tally_change({ state, number }) {
            this.state = state;
            this.number = number;
        },
        prod_preview_program_change(data) {
            console.log(data);
            this.producerClientKey = data.clientSource;
            this.producerPreviewScene = data.previewScene;
            this.producerProgramScene = data.programScene;

            if (this.targetsMe(this.producerProgramScene)) {
                this.state = "active";
            } else if (this.targetsMe(this.producerPreviewScene)) {
                this.state = "preview";
            } else {
                this.state = "inactive";
            }
        }
    },
    computed: {
        producer() {
            if (!this.producerClientKey) return null;
            return ReactiveRoot(`client-${this.producerClientKey}`);
        },
        liveMatch() {
            const matchID = this.client?.broadcast?.live_match?.[0];
            if (!matchID) return null;
            return ReactiveRoot(matchID, {
                player_relationships: ReactiveArray("player_relationships"),
                teams: ReactiveArray("teams", {
                    theme: ReactiveThing("theme")
                })
            });
        },
        teams() {
            if (!this.liveMatch?.teams?.length) return [];
            const teams = this.liveMatch?.teams;
            if (this.liveMatch.flip_teams) return teams.reverse();
            return teams;
        },
        scores() {
            if (!this.liveMatch) return [null, null];
            const scores = [this.liveMatch.score_1, this.liveMatch.score_2];
            if (this.liveMatch.flip_teams) return scores.reverse();
            return scores;
        },
        selfObserverNumber() {
            return (this.liveMatch?.player_relationships || []).filter(rel => rel.singular_name === "Observer").findIndex(rel => rel.player?.[0] === this.client?.staff?.[0]) + 1;
        }
    },
    methods: {
        async getWakeLock() {
            this.wakeLock = await navigator.wakeLock.request();
            this.wakeLock.addEventListener("release", () => {
                console.log("Screen Wake Lock released:", this.wakeLock.released);
            });
            console.log("Screen Wake Lock released:", this.wakeLock.released);
        },
        targetsMe(sceneName) {
            const number = this.number || this.selfObserverNumber;
            if (!number) return false;
            return ["Obs", "Game"].some(str => sceneName.toLowerCase().includes(str.toLowerCase())) && sceneName.includes(number.toString());
        }

    },
    async mounted() {
        if ("wakeLock" in navigator) {
            // screen will stay on in supported browsers
            await this.getWakeLock();
            document.addEventListener("visibilitychange", async () => {
                if (this.wakeLock !== null && document.visibilityState === "visible") {
                    await this.getWakeLock();
                }
            });
        }
    },
    data: () => ({
        state: "inactive",
        number: null,
        wakeLock: null,
        producerClientKey: null,
        producerPreviewScene: null,
        producerProgramScene: null,
        showProducerInfo: true,

        noBroadcastStyle: true,
        noStinger: true
    }),
    metaInfo() {
        return {
            title: `Tally Viewer | ${this.client?.name || this.client?.key || ""}`
        };
    }
};
</script>

<style scoped>
.tally-block {
  height: 100vh;
  width: 100vw;
  background-color: #000000;
  color: #ffffff;
  display: grid;
  place-items: center;
  font-size: clamp(10px, 8vw, 25vh);
  transition: background-color 0.2s ease;
  font-family: "SLMN-Industry", "Industry", sans-serif;
}

.state {
  font-size: 1.25em;
}

.metadata {
  font-size: .75em;
}


.tally-block.preview {
  background-color: #00ff00;
  color: #000000;
}

.tally-block.active,
.tally-block.program {
  background-color: #ff0000;
}

.prod-info, .team-info {
    display: flex;
    text-align: center;
    font-size: 0.4em;
    background-color: rgba(32,32,32,0.5);
    color: white;
    padding: .5em 1em;
    width: 100%;
}

.team-info {
    padding: 0;
}
.team-name {
    padding: .5em 1em;
}
.team-score {
    padding: .2em 1em;
}

.team {
    display: flex;
    width: 50%;
    justify-content: center;
    align-items: center;
}
.team-info .team:nth-child(2) {
    flex-direction: row-reverse;
}
.team-name {
    flex-grow: 1;
    text-align: center;
    line-height: 1.2;
}
.team-logo {
    height: 100%;
    width: 3em;
}

.prod-name {
    font-weight: bold;
    padding: 0.1em .5em;
}
.prod-scenes {
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 0.8em;
    flex-grow: 1;
    margin-left: 1em;
    font-weight: bold;
}
.prod-scenes div {
    border: 1px solid rgba(255,255,255,0.5);
    padding: 0.5em .25em;
    margin: 0 0.25em;
    background-color: black;
    width: 100%;
    line-height: 1.2;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    text-align: center;
}

.prod-scenes .prod-preview {
    color: lime;
    border-color: lime;
    border-radius: .1em;
}
.prod-scenes .prod-program {
    color: #ff4646;
    border-color: #ff0000;
    border-radius: .1em;
}

.tally-overlay {
    position: absolute;
    width: 100vw;
    height: 100vh;
    top: 0;
    left: 0;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
}

.first-to {
    order: 2;
    padding: .5em .25em;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    text-align: center;
    font-weight: bold;
}
</style>

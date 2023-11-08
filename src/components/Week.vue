<script>
    import axios from 'axios';

    export default {
        name: "App",
        data() {
            return {
                activeWeekButton: 0,
                currentWeekButton: 0,
                mainData: {
                    "table": {
                        "type": "",
                        "name": "",
                        "week" : 0,
                        "group": "",
                        "table": [],
                        "link": ""
                    },
                    "weeks": []
                },
            }
        },
        methods: {
            async request(link) {
                const response = await axios.get(link)
                return response.data;
            },

            async loadData(link, dest) {
                let answer = await this.request(link);
                if (dest === 'main') { this.mainData = answer; }
                else if (dest === 'sec') {
                    this.mainData.table.table[3] = answer.table.table[3];
                    this.mainData.table.table[6] = answer.table.table[6];
                };
            },

            async getDataFromApi(prim, sec, week = 0) {
                if (week === 0) {
                    await this.loadData(`https://webictis.sfedu.ru/schedule-api/?group=${prim}.html`, 'main');
                    await this.loadData(`https://webictis.sfedu.ru/schedule-api/?group=${sec}.html`, 'sec');
                    this.activeWeekButton = this.mainData.table.week;
                    this.currentWeekButton = this.mainData.table.week;
                } else {
                    this.activeWeekButton = week;
                    await this.loadData(`https://webictis.sfedu.ru/schedule-api/?group=${prim}.html&week=${week}`, 'main');
                    await this.loadData(`https://webictis.sfedu.ru/schedule-api/?group=${sec}.html&week=${week}`, 'sec');
                }
            },
        },
        mounted() {
            this.getDataFromApi(125, 191);
        }
    }
</script>

<template>
    <div><b>{{ `${mainData.table.type} ${mainData.table.name}` }}</b></div>

    <div class="weekButtonsBlock">
        <button :class="[
                item === activeWeekButton ? 'activeWeek' : '',
                item === currentWeekButton ? 'currentWeek' : '',
                'generalWeek'
                ]" v-on:click="getDataFromApi(125, 191, item)" v-for="item in mainData.weeks">{{ item }}</button>
    </div>

    <table class="lessonsTable">
        <tr>
            <td class="infoCell" v-for="item in mainData.table.table[0]"><div class="headingCell">{{ item }}</div></td>
        </tr>
        <tr>
            <td class="infoCell" v-for="item in mainData.table.table[1]"><div class="headingCell">{{ item }}</div></td>
        </tr>
        <tr v-for="(lesson, lessonIndex) in mainData.table.table">
            <td :class="[
                item.includes('лек.') ? 'lessonLect' : '',
                item.includes('лаб.') ? 'lessonLab' : '',
                item.includes('пр.') ? 'lessonPr' : '',
                index === 0 ? 'infoCell' : ''
                ]" v-if="lessonIndex > 1" v-for="(item, index) in lesson"><div class="lessonCell">{{ item }}</div></td>
        </tr>
    </table>
</template>
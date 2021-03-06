<template>
    <div class="row">
        <div v-if="loading && $i18n">{{ $t('generic.loading')}}...</div>
        <div v-else-if="loading">Loading ...</div>

        <div v-if="error" class="error"></div>
        <div class="col-sm-12">
            <div class="card">
                <div class="card-header">
                    <h2 v-if="$i18n">{{$t('generic.calender')}}</h2>
                    <h2 v-else>Calendar</h2>
                </div>

                <div class="card-block">
                    <div class="row">
                        <div class="col-sm-12">
                            <CalendarHeader :current-month="currentMonth"
                                       :first-day="firstDay"
                                       :locale="appLocale">
                            </CalendarHeader>

                            <div class="full-calendar-body">
                                <div class="weeks">
                                    <strong class="week" v-for="(dayIndex, index) in 7" :key="index">{{ (dayIndex - 1) | weekDayName(firstDay, appLocale) }}</strong>
                                </div>

                                <div class="dates" ref="dates">
                                    <Week v-for="(week, index) in Weeks"
                                          :firstDay="firstDay"
                                          :key="week + index"
                                          :week="week"
                                          :canAddEvent="canAddEvent"
                                          :canDeleteEvent="canDeleteEvent"
                                          :displayWeekNumber="displayWeekNumber"
                                          @eventAdded="eventAdded"
                                          @eventDeleted="eventDeleted">
                                    </Week>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
    import moment from 'moment';
    import {CHANGE_MONTH, EVENT_ADDED, EVENT_DELETED} from './actions';

    export default {
        components: {
            'CalendarHeader': require('./Components/Header.vue').default ? require('./Components/Header.vue').default : require('./Components/Header.vue'),
            'Week': require('./Components/Week.vue').default ? require('./Components/Week.vue').default : require('./Components/Week.vue')
        },
        props: {
            allEvents: {
                type: Array,
                default: function () {
                    return [];
                }
            },

            firstDay: {
                type: Number | String,
                validator (val) {
                    let res = parseInt(val);
                    return res >= 0 && res <= 6;
                },
                default: 0
            },

            displayWeekNumber: {
                type: Boolean,
                default: true,
            },

            canAddEvent: {
                type: Boolean,
                default: true
            },

            canDeleteEvent: {
                type: Boolean,
                default: true
            },
        },

        data () {
            return {
                loading: true,
                error: null,
                currentMonth: moment().startOf('month'),
            }
        },

        created () {
            let me = this;
            this.$root.$on(CHANGE_MONTH, function (payload) {
                me.currentMonth = payload;
            });      
        },

        mounted () {
            this.loading = false;
        },

        computed: {
            Weeks () {
                let monthMomentObject = this.getMonthViewStartDate(this.currentMonth, this.firstDay);
                let weeks = [], week = [], day = null, weekIndex, dayIndex;

                for (weekIndex=0; weekIndex < 5; weekIndex++) {
                    week = [];

                    for (dayIndex=0; dayIndex < 7; dayIndex++) {
                        day = {
                            isToday: monthMomentObject.isSame(moment(), 'day'),
                            isCurrentMonth: monthMomentObject.isSame(this.currentMonth, 'month'),
                            weekDay: dayIndex,
                            isWeekEnd: (dayIndex == 5 || dayIndex == 6),
                            date: moment(monthMomentObject),
                            events: this.getEvents(monthMomentObject)
                        };

                        week.push(day);

                        monthMomentObject.add(1, 'day');
                    }
                    weeks.push(week);
                }

                return weeks;
            },

            appLocale : function () {
                if(typeof i18n != 'undefined')
                    return i18n.locale;
                
                return 'en';
            },

            events: function () {
                return this.allEvents;
            }
        },
        
        methods: {
            getEvents (date) {
                return this.events.filter(event => {
                    return date.isSame(event.date, 'day') ? event : null;
                });
            },

            getMonthViewStartDate (date, firstDay) {
                firstDay = parseInt(firstDay);

                let start = moment(date).locale(this.appLocale);    
                let startOfMonth = moment(start.startOf('month'));

                start.subtract(startOfMonth.day(), 'days');

                if (startOfMonth.day() < firstDay) {
                    start.subtract(7, 'days');
                }

                start.add(firstDay, 'days');

                return start;
            },

            eventAdded(event) {
                this.$emit(EVENT_ADDED, event);
            },

            eventDeleted(event) {
                this.$emit(EVENT_DELETED, event);
            }
        },
        filters: {
            weekDayName (weekday, firstDay, locale) {
                firstDay = parseInt(firstDay);
                const localMoment = moment().locale(locale);
                return localMoment.localeData().weekdaysShort()[(weekday + firstDay) % 7];
            }
        }
    }

</script>
<style>
    ul, p {
        margin: 0;
        padding: 0;
    }

    .full-calendar-body {
        margin-top: 20px;
    }
    .weeks {
        display: flex;
        border-top: 1px solid #e0e0e0;
        border-bottom: 1px solid #e0e0e0;
        border-left: 1px solid #e0e0e0;
    }
    .week {
        flex: 1;
        padding: 5px;
        text-align: center;
        border-right: 1px solid #e0e0e0;
    }

    .dates {
        position: relative;
    }
</style>
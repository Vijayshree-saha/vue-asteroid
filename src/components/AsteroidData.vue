<template>
    <div class="row">
        <div class="col-12">
            <form>
                <div class="form-group">
                    <label for="exampleInputEmail1">Start date</label>
                    <datepicker v-model="startDate" name="uniquename" format="yyyy-MM-dd" input-class="form-control" @closed="startDate=customFormatter(startDate)"></datepicker>
                </div>
                <div class="form-group">
                    <label for="exampleInputPassword1">End date</label>
                    <datepicker v-model="endDate" name="uniquename" format="yyyy-MM-dd" input-class="form-control" @closed="endDate=customFormatter(endDate)"></datepicker>
                </div>
                <button type="submit" class="btn btn-primary" @click.prevent="getnearAsteriod()">Submit</button>
            </form>
            <p v-if="stats !== null">Fastest Asteroid: <b>id:</b> {{ stats.fastest.id }} <b>Speed:</b> {{ stats.fastest.speed }} </p>
            <p v-if="stats !== null">Closest Asteroid: {{ stats.closestItem }}</p>
            <p v-if="stats !== null">Average Size: {{ stats.averageKilo }}</p>
        </div>
        <div class="col-12" style="height: 200px;">
            <canvas id="myChart" width="200" height="200"></canvas>
        </div>
    </div>
</template>

<script>
import Vue from 'vue';
import Datepicker from 'vuejs-datepicker';
import axios from 'axios';
import moment from 'moment';

export default {

    data() {
        return {
            startDate: '2020-01-01',
            endDate: '2020-01-05',
            nearObjects: null,
        }
    },

    components: {
        Datepicker,
    },

    methods: {
        getnearAsteriod() {
            //Passed variable in string.
            let url = `https://api.nasa.gov/neo/rest/v1/feed?start_date=${this.startDate}&end_date=${this.endDate}&api_key=gDmwsft1TRn0PqCpuequ7dqRnEhFarI9KRQwR1TY`;
            axios.get(url)
            .then(response => {
                this.nearObjects = response.data.near_earth_objects;
                this.createChart('myChart');
                this.$toasted.success('Successfully Created Graph', {
                    //theme of the toast you prefer
                    theme: 'toasted-primary',
                    //position of the toast container
                    position: 'top-left',
                    //display time of the toast
                    duration: 5000
                }); 
            })
            .catch(error => {
                this.$toasted.error(error.response.data.error_message, {
                    //theme of the toast you prefer
                    theme: 'bubble',
                    //position of the toast container
                    position: 'top-right',
                    //display time of the toast
                    duration: 5000
                });                 
            });
        },
        createChart(chartId) {
                const ctx = document.getElementById(chartId);
                const myChart = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels:  this.getchartLables,
                    datasets: [{
                        label: 'Number of Plants',
                        data: this.totalplanetIndays,
                        backgroundColor: [
                            'rgba(255, 99, 132, 0.2)',
                            'rgba(54, 162, 235, 0.2)',
                            'rgba(255, 206, 86, 0.2)',
                            'rgba(75, 192, 192, 0.2)',
                            'rgba(153, 102, 255, 0.2)'
                        ],
                        borderColor: [
                            'rgba(255, 99, 132, 1)',
                            'rgba(54, 162, 235, 1)',
                            'rgba(255, 206, 86, 1)',
                            'rgba(75, 192, 192, 1)',
                            'rgba(153, 102, 255, 1)'
                        ],
                        borderWidth: 1
                    }]
                },
                options: {
                    scales: {
                        yAxes: [{
                            ticks: {
                                beginAtZero: true
                            }
                        }]
                    }
                }
            });
        },
        customFormatter(date) {
            return moment(date).format('yyyy-MM-DD');
        }
    },
    computed: {

        stats () {
            if (! this.nearObjects) {
                return null;
            }
            // Below three function converted into a single funtion
            let speeds = [];
            let cloest = [];
            let average = [];
            let total = 0;
            
            // Loop over object value
            Object.keys(this.nearObjects).forEach((key, index) => {
                const dayObjects = this.nearObjects[key];

                dayObjects.forEach((obj) => {
                    speeds.push({
                        id: obj.id,
                        speed: obj.close_approach_data[0].relative_velocity.kilometers_per_hour
                    });
                    cloest.push(obj.close_approach_data[0].miss_distance.kilometers);
                    average.push(obj.estimated_diameter.kilometers.estimated_diameter_max);
                })
                for(var i = 0; i < average.length; i++) {
                    total += average[i];
                }
            })
            //Get the max value out of the array
            const getMax = Object.values(speeds).reduce((prev, current) => (speeds[prev] > speeds[current]) ? prev : current);

            return {
                // Below three function converted into a single funtion as the middle code is repeatating.
                fastest: getMax,
                closestItem: Math.min.apply(Math, cloest),
                averageKilo: total / average.length,
            }
        },
        getchartLables() {
            if (! this.nearObjects) {
                return null;
            }
            const propertyNames = Object.keys(this.nearObjects);
            return propertyNames;
        },

        totalplanetIndays() {
            if (! this.nearObjects) {
                return null;
            }
            let propertyCount = [];
            Object.keys(this.nearObjects).forEach((key, index) => {
                //console.log(key, this.nearObjects[key]);
                const dayObjects = this.nearObjects[key];
                
                propertyCount.push(dayObjects.length);
                
            })
            return propertyCount;
        }
    }
}
</script>
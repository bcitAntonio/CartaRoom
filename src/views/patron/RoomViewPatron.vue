<template>
<div class="editRoom">
    <div class="banner">
        <br>
        <p class="banner_text">Room View</p>
    </div>
    <div class="rowPicture">
        <div id="rowBigPic">
            <div id="bigPicture">
                <img src="../../assets/banner/image.png">
                </div>
            </div>
            <div id="rowSmallPic">
                <div class="smallPicture">
                    <img src="../../assets/banner/image.png">
                </div>
                    <div class="smallPicture">
                        <img src="../../assets/banner/image.png">
                </div>
                        <div class="smallPicture">
                            <img src="../../assets/banner/image.png">
                </div>
                        </div>

                    </div>
                    <div id="view">
                        <div class="row">
                            <div class="column1">
                                <label>Room Name</label>
                            </div>
                            <div class="column2">
                                <p class="info"> {{ roomInfo.name }}</p>
                            </div>
                        </div>
                        <div class="row">
                            <div class="column1">
                                <label>Capacity</label>
                            </div>
                            <div class="column2">
                                <p class="info2"> {{ roomInfo.capacity }} People</p>
                            </div>
                        </div>
                        <div class="row">
                            <div class="column1">
                                <label>Description</label>
                            </div>
                            <div class="column2">
                                <p class="info"> {{ roomInfo.description }}</p>
                            </div>
                        </div>
                        <div class="row">
                            <div class="column1">
                                <label>Address</label>
                            </div>
                            <div class="column2">
                                <p class="info"> {{ roomInfo.address }}</p>
                            </div>
                        </div>
                        <div class="row">
                            <div class="column1">
                                <label>Business Hours</label>
                            </div>
                            <div class="column2">
                                <p class="info2"> {{ roomInfo.openTime }} - {{ roomInfo.closeTime }}</p>
                            </div>
                        </div>

                        <div class="row">
                            <div class="column1">
                                <label>Amenities</label>
                            </div>
                            <div class="column2">
                                <ul class="info2" v-for="amenity in roomInfo.amenities" v-bind:key="amenity['.key']">
                                    {{amenity}}
                                </ul>
                            </div>
                        </div>

                        <div class="row">
                            <div id="line">
                                <hr>
                            </div>
                        </div>

                        <div class="row">
                            <div class="column1">
                                <label>Date</label>
                            </div>
                            <div class="column2">
                                <input id="info_date" @keyup.enter="doMath" v-focus type="date" v-model="date"><br />
                                <h3>{{msg}}</h3>
                                <ul id="time_list" v-for="index in timeSlotsAvailable" :key="index">
                                    <p id="time_slots">{{roomInfo.bookingSlots[index].startingTime}}:00 to {{roomInfo.bookingSlots[index].endingTime}}:00
                                        <button v-on:click="bookRoom(roomInfo.bookingSlots[index].startingTime, roomInfo.bookingSlots[index].endingTime)">Book Room</button>
                                    </p>
                                </ul>
                                <v-btn id="timeButton" @click="doMath"><b><u>{{msg1}}</u></b></v-btn>
                            </div>
                        </div>
                        <!-- <p>{{roomInfo.bookingSlots[1].startingTime}}:00 to {{roomInfo.bookingSlots[1].endingTime}}:00</p> <br /> -->
                        <!-- </li> -->
                    </div>
                    <router-link to="/search">Cancel</router-link>
                </div>
</template>

<script>
import firebase from 'firebase';
import db from '@/firebase.js';

//alert styling
import Vue from 'vue'
import 'v-slim-dialog/dist/v-slim-dialog.css'
import SlimDialog from 'v-slim-dialog'
Vue.use(SlimDialog)

var createRoom = firebase.functions().httpsCallable('createRoom');
var roomViewPatronCreated = firebase.functions().httpsCallable('roomViewPatronCreated');
var checkBookingExist = firebase.functions().httpsCallable('checkBookingExist');
var getHostNameRoomViewPatron = firebase.functions().httpsCallable('getHostNameRoomViewPatron');
var updateUIDBookingViewRoomPatron = firebase.functions().httpsCallable('updateUIDBookingViewRoomPatron');
var updateUIDAllBookingViewRoomPatron = firebase.functions().httpsCallable('updateUIDAllBookingViewRoomPatron');

var userID;
var roomID = '1';
var storageRef = firebase.storage().ref();

firebase.auth().onAuthStateChanged(function (user) {
    if (user) {
        // console.log(user.uid); //a@a.com = gbEw7s5ic1drxG3vgFWD3DAMb972
        userID = user.uid;
    } else {
        // console.log("No user available"); 
        userID = 'null';
    }
});

export default {
    name: 'addRoom',

    data() {
        return {
            id: 0,
            room: {},
            timeSlotsAvailable: 0,
            msg: '',
            msg1: 'View time slots',
            date: '',
            hostName: '',
            userEmail: '',
            amenities: [{
                    offering: 'wifi'
                },
                {
                    offering: 'projector'
                },
                {
                    offering: 'whiteboard'
                },
                {
                    offering: 'Ethernet'
                },
            ],
            roomInfo: {
                hostID: '',
                name: '',
                capacity: '',
                description: '',
                address: '',
                roomID: roomID,
                reserved: 'false',
                bookingCounter: 0,
                openTime: 0,
                closeTime: 0,
                amenities: [],
                bookingSlots: [{
                    startingTime: '',
                    endingTime: ''
                }]
            }
        }
    },

async created() {
        this.id = this.$route.params.id;

        await createRoom({
            id: this.id,
            roomInfo: this.roomInfo
        }).then((result) => {
            this.roomInfo = result.data.roomInfo
        }).catch(function (error) {
            console.log(error);
        });

        await roomViewPatronCreated({
            userEmail: this.userEmail
        }).then((result) => {
            this.userEmail = result.data.userEmail;
        }).catch(function (error) {
            console.log(error);
        });
    },

    methods: {
        doMath: function () {
            var startHoursMinutes = this.roomInfo.openTime.split(/[.:]/);
            var startHours = parseInt(startHoursMinutes[0], 10);
            var closeHoursMinutes = this.roomInfo.closeTime.split(/[.:]/);
            var closeHours = parseInt(closeHoursMinutes[0], 10);
            this.timeSlotsAvailable = closeHours - startHours;
            this.msg = 'Time Slots';
            this.msg1 = '';
        },

        bookRoom: async function (startTime, endTime) {
            var userInfo = firebase.auth().currentUser.uid;
            var uniqueKeyIDBooking = '1';
            var elseStatement = false;
            var snapshotExist = false;
            var initialBookingID = '1';
            var dateChecked;

            //This creates a string with all the neccessary info needed to see if a time slot is already taken or not
            var bookingInfo = startTime.toString() + '->' + endTime.toString() + '->' + this.date + '->' + this.roomInfo.roomID;

            //if date is null alter message "please select a date before moving on"
            //if date is seleted, check booking time slot is available in datebase and return a boolean snapshotExist from cloud function 
            if (this.date === '') {
                this.$dialogs.alert('Please select a date before moving on', {
                    title: 'Warning!',
                    okLabel: 'OK, BOSS'
                });
                dateChecked = false;
            } else {
                await checkBookingExist({
                    bookingInfo: bookingInfo
                }).then((result) => {
                    snapshotExist = result.data.snapshotExist;
                    console.log("snapshotExist: " + snapshotExist);
                }).catch(function (error) {
                    console.log(error);
                });
                dateChecked = true;
            }

            //if snapshotExist is true, meaning booking slot already exists in datebase alter message "not avaiable"
            //if snapshotExist is false, book selected time slot and update database using cloud function
            if (snapshotExist) {
                var msgx = 'TIME SLOT NOT AVAILABLE FOOOR:\n' + 'date: ' + this.date + '\n' + "time: " + startTime + ':00' + " - " + endTime + ':00\n' + 'PLEASE SELECT ANOTHER DATE/TIME';
                this.$dialogs.alert(msgx, {
                    title: 'Warning!',
                    okLabel: 'OK'
                });
            } else {
                await getHostNameRoomViewPatron({
                    hostID: this.roomInfo.hostID,
                    hostName: this.hostName
                }).then((result) => {
                    console.log("this.roomInfo.hostID: " + this.roomInfo.hostID);
                    console.log("hostName: " + result.data.hostName);
                    this.hostName = result.data.hostName
                }).catch(function (error) {
                    console.log(error);
                });
                elseStatement = true;
            }

            //update database if snapshotExist is false
            if (elseStatement && dateChecked) {
                const info = {
                    room: this.roomInfo,
                    user: userInfo,
                    userEmail: this.userEmail,
                    bookingID: '1',
                    startTime: startTime,
                    endTime: endTime,
                    date: this.date,
                    host: this.hostName,
                    bookingInfo: bookingInfo,
                    status: 'not completed'
                }

                await updateUIDBookingViewRoomPatron({
                    info: info,
                    uniqueKeyIDBooking: uniqueKeyIDBooking
                }).then((result) => {
                    initialBookingID = result.data.uniqueKeyIDBooking;
                }).catch(function (error) {
                    console.log(error);
                });

                const info2 = {
                    room: this.roomInfo,
                    user: userInfo,
                    userEmail: this.userEmail,
                    allbookingID: '1',
                    initialBookingID: initialBookingID,
                    startTime: startTime,
                    endTime: endTime,
                    date: this.date,
                    host: this.hostName,
                    bookingInfo: bookingInfo,
                    status: 'not completed'
                }

                await updateUIDAllBookingViewRoomPatron(info2).then(() => {
                    var msgx = 'BOOKING CONFIRMED\n' + 'Booking Details:\n' + 'date: ' + this.date + '\n' + "time: " + startTime + ':00' + " - " + endTime + ':00';
                    this.$dialogs.alert(msgx, {
                        title: 'Warning!',
                        okLabel: 'OK'
                    });
                    this.$router.push({
                        path: '/currentBookingsPatron'
                    });
                })
            }
        }
    },
}
</script>

<style scoped>
#rowSmallPic {
    display: flex;
    align-items: center;
    width: 50%;
    padding: 10px 25%;
}

#bigPicture {
    margin-top: 1%;
}

.smallPicture {
    width: 10%;
    flex: 33%;
}

.row {
    display: flex;
}

.column1 {
    width: 40%;
}

.column1 label {
    float: right;
    padding-right: 10%;
}

.column2 {
    width: 60%;
    padding: 0;
}

.column2 ul {
    display: inline-block;
    margin-right: 20px;
    margin-top: 10px;
    margin-bottom: 0;
    vertical-align: middle;
}

.smallPicture img {
    width: 100%;
}

.info {
    border: 0.75px solid darkgrey;
    width: 75%;
    font-size: 20px;
    line-height: 35px;
    box-sizing: border-box;
    background: #FFFFFF;
    border-radius: 5px;
    margin-right: 100px;
    word-break: break-word;

}

.info2 {
    border: 0.75px solid darkgrey;
    width: 160px;
    padding-left: 0;
    font-size: 20px;
    line-height: 35px;
    box-sizing: border-box;
    background: #FFFFFF;
    border-radius: 5px;
    word-break: break-word;
}

label {
    margin-left: 30%;
    margin-top: 20px;
    margin-bottom: 15px;
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    font-style: normal;
    font-weight: normal;
    font-size: 25px;
    line-height: 35px;
    display: flex;
    align-items: center;
    color: #000000;
}

.banner_text {
    font-family: Rajdhani;
    font-style: normal;
    font-weight: 600;
    font-size: 90px;
    line-height: 191px;
    text-align: center;
    color: #000000;
}

.banner {
    height: 400px;
    width: 100%;
    background: linear-gradient(rgba(255, 255, 255, .5), rgba(255, 255, 255, .5)), url("../../assets/banner/Host2.jpg");
    background-repeat: no-repeat;
    background-size: cover;
    margin-top: -150px;
    padding-top: 150px;
}

button {
    background: #FFFFFF;
    border-radius: 15px;
    height: 44px;
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    font-style: normal;
    font-weight: normal;
    font-size: 18px;
    line-height: 35px;
    text-align: center;
    width: 177px;
    color: #000000;
    margin: 0px 20px;

}

input {
    margin: 10px 0;
    width: 20%;
    padding: 15px;
}

span {
    display: block;
    margin-top: 20px;
    font-size: 11px;
}

h3 {
    margin-bottom: 0;
}

#view {
    margin: 40px auto 45px auto;
    padding: 1% 0;
    background: rgba(218, 229, 227, 0.9);
    border-radius: 15px;
    width: 55%;
    height: 60%;
}

#info_date {
    width: 30%;
}

#time_slots {
    margin-top: 0;
    margin-bottom: 0;
}

#time_slots button {
    width: 30%;
}

#time_list {
    width: 90%;
}
</style>

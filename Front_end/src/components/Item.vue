<template>
    <div class="api-body">
        <h2>API Integration</h2>
        <!-- menu bar -->
        <div class="select-bar">
            <div class="left-section">
                <!-- chargebee -->
                <select :style="chargebeeFlag ? 'display: block' : 'display: none'" ref="chargebee" v-model="actions" @change="onChange($event)">
                    <option value="" selected>All Members</option>
                    <option value="current_basic">Basic Members</option>
                    <option value="current_premier">Premier Members</option>
                    <option value="new_monthly_basic_purchase">New Monthly Basic Puschases</option>
                    <option value="monthly_basic_cancelled">Monthly Basic Cancellations</option>
                    <option value="monthly_premier_purchase">Monthly Premier Purchases</option>
                    <option value="premier_schedule">Premier Schedule</option>
                </select>

                <!-- activecampaign -->
                <select :style="!chargebeeFlag ? 'display: block' : 'display: none'" ref="activecampaign" v-model="actions" @change="onChangeActive($event)">
                    <option value="" selected>All Members</option>
                    <option value="current_group">Current Group Members</option>
                    <option value="current_xypn">Current XYPN Members</option>
                    <option value="current_complimentary">Current Complimentary Members</option>
                    <option value="premier_yesterday">Premier Auto-Renewed Month to Yesterday</option>
                </select>

                <Datepicker
                    v-model="date"
                    clear-button-icon="fa fa-times"
                    calendar-button-icon="fa fa-calendar"
                    :calendar-button="true"
                    :clear-button="true"
                    :placeholder="'When?'"
                    @selected="changeDate"
                />
            </div>

            <div class="right-section">
                <!-- select section -->
                <div class="select-area">
                    <button @click="changeStyle(1)" :style="(chargebeeFlag === true) ? 'background: #ABE5C4': ''">Chargebee</button>
                    <button @click="changeStyle(-1)" :style="(chargebeeFlag === false) ? 'background: #ABE5C4': ''">ActiveCampaign</button>
                </div>

                <downloadexcel
                    class   = "btn-default download-btn"
                    :data   = "json_data"
                    :fields = "(chargebeeFlag) ? chargebee_json_fields : active_json_fields"
                    worksheet = "My Worksheet"
                    type="csv"
                    :before-generate = "startDownload"
                    :name    = "filename + '.xls'">
                    <mdb-icon icon="arrow-down" title="download" />
                </downloadexcel>
            </div>
        </div>

        <div v-if="currentTitle !== ''">
            <h2>{{ currentTitle }}&nbsp;:&nbsp;{{ number }}</h2>
        </div>
        <!-- chargebee -->
        <Chargebee
            :data="chargebee_data"
            :visibleFlag="chargebeeFlag"
        />

        <!-- activecampaign -->
        <ActiveCampaign
            :data="active_data"
            :visibleFlag="!chargebeeFlag"
        />

        <!-- loading section -->
        <div class="loading-area" v-if="loadingFlag">
            <span>Now, So many data is loading.</span><br>
            <span>Just a moment...</span>
        </div>
    </div>
</template>

<script>
import Datepicker from 'vuejs-datepicker';
import { api } from '../services/api';
import downloadexcel from "vue-json-excel";
import moment from 'moment';
import Chargebee from './Chargebee';
import ActiveCampaign from './ActiveCampaign';
import { mdbIcon } from 'mdbvue';

let now = new Date();
let months = ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"];

export default {
  name: 'Item',
  components: {
    downloadexcel,
    Datepicker,
    Chargebee,
    ActiveCampaign,
    mdbIcon
  },
  data () {
    return {
        actions: "",
        activeFlag: false,
        currentTitle: "",
        number: "",
        chargebeeData: null,
        activecampaignData: null,
        chargebee_data: null,
        active_data: null,
        loadingFlag: true,
        filename: "All_Members",
        date: null,
        chargebeeFlag: null,
        chargebee_json_fields: {
            'Email': 'id',
            'Recurring Items': 'recurring',
            'Status': 'status',
            'Created At': 'created_at',
            'Next Billing At': 'next_billing'
        },
        active_json_fields: {
            'Full Name': 'name',
            'Email': 'email',
            'Phone': 'phone',
            'Account': 'account',
            'Date Created': 'date_created'
        },
        json_data: null,
        json_meta: [
            [
                {
                    'key': 'charset',
                    'value': 'utf-8'
                }
            ]
        ]
    };
  },
  methods: {
    startDownload: function () {
        if (this.activeFlag) {
            if (this.json_data.length === 0) {
                window.alert('There is no data to download!!!');
            } else {
                window.alert('Start Downloading');
            }
        } else {
            window.alert('Now loading data, Just a moment');
        }
    },
    onChange: function (event) {
        let tables = [];
        // search by select
        if (this.activeFlag) {
            if (this.date === null) {
                switch (event.target.value) {
                    case "current_basic":
                        this.chargebeeData.forEach(item => {
                            if (item.subscription.billing_period_unit !== "year") {
                                if(item.subscription.status === "active" || item.subscription.status === "non_renewing") {
                                    let tableItem = {};
                                    tableItem["id"] = item.customer.email;
                                    tableItem["status"] = item.subscription.status.toUpperCase();
                                    tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                    tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                    tableItem["recurring"] = "Basic Member";
                                    tables.push(tableItem);
                                }
                            }
                        });
                        this.json_data = tables;
                        this.filename = "Current_Basic";
                        this.currentTitle = "Current Basic Members";
                        this.number = tables.length !== 0 ? tables.length : "No data";
                        break;
                    case "current_premier":
                        this.chargebeeData.forEach(item => {
                            if (item.subscription.billing_period_unit === "year") {
                                if(item.subscription.status === "active" || item.subscription.status === "non_renewing") {
                                    let tableItem = {};
                                    tableItem["id"] = item.customer.email;
                                    tableItem["status"] = item.subscription.status.toUpperCase();
                                    tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                    tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                    tableItem["recurring"] = "Premier Member";
                                    tables.push(tableItem);
                                }
                            }
                        });
                        this.json_data = tables;
                        this.filename = "Current_Premier";
                        this.currentTitle = "Current Premier Members";
                        this.number = tables.length !== 0 ? tables.length : "No data";
                        break;
                    case "new_monthly_basic_purchase":
                        this.chargebeeData.forEach(item => {
                            if (item.subscription.billing_period_unit !== "year" && (now.getMonth() === moment(item.subscription.created_at * 1000).toDate().getMonth() && now.getYear() === moment(item.subscription.created_at * 1000).toDate().getYear())) {
                                let tableItem = {};
                                tableItem["id"] = item.customer.email;
                                tableItem["status"] = item.subscription.status.toUpperCase();
                                tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                tableItem["recurring"] = "Basic Member";
                                tables.push(tableItem);
                            }
                        });
                        this.json_data = tables;
                        this.filename = "New_Monthly_Basic_Purchase";
                        this.currentTitle = "New Monthly Basic Purchase Members";
                        this.number = tables.length !== 0 ? tables.length : "No data";
                        break;
                    case "monthly_basic_cancelled":
                        this.chargebeeData.forEach(item => {
                            if (item.subscription.billing_period_unit !== "year" && item.subscription.status === "cancelled" && item.subscription.billing_period === 1) {
                                let tableItem = {};
                                tableItem["id"] = item.customer.email;
                                tableItem["status"] = item.subscription.status.toUpperCase();
                                tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                tableItem["recurring"] = "Basic Member";
                                tables.push(tableItem);
                            }
                        });
                        this.json_data = tables;
                        this.filename = "Monthly_Basic_Cancelled";
                        this.currentTitle = "Monthly Basic Cancellations";
                        this.number = tables.length !== 0 ? tables.length : "No data";
                        break;
                    case "monthly_premier_purchase":
                        this.chargebeeData.forEach(item => {
                            if (item.subscription.billing_period_unit === "year" && item.subscription.status === "cancelled" && item.subscription.billing_period === 1) {
                                let tableItem = {};
                                tableItem["id"] = item.customer.email;
                                tableItem["status"] = item.subscription.status.toUpperCase();
                                tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                tableItem["recurring"] = "Premier Member";
                                tables.push(tableItem);
                            }
                        });
                        this.json_data = tables;
                        this.filename = "Monthly_Premier_Purchase";
                        this.currentTitle = "Monthly Premier Purchase Members";
                        this.number = tables.length !== 0 ? tables.length : "No data";
                        break;
                    case "premier_schedule":
                        this.chargebeeData.forEach(item => {
                            if (item.subscription.billing_period_unit === "year" && item.subscription.billing_period === 1) {
                                let tableItem = {};
                                tableItem["id"] = item.customer.email;
                                tableItem["status"] = item.subscription.status.toUpperCase();
                                tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                tableItem["recurring"] = "Premier Member";
                                tables.push(tableItem);
                            }
                        });
                        this.json_data = tables;
                        this.filename = "Premier_Schedule";
                        this.currentTitle = "Premier Schedule Members";
                        this.number = tables.length !== 0 ? tables.length : "No data";
                        break;
                    case "":
                        this.chargebeeData.forEach(item => {
                            let tableItem = {};
                            tableItem["id"] = item.customer.email;
                            tableItem["status"] = item.subscription.status.toUpperCase();
                            tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                            tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                            tableItem["recurring"] = (item.subscription.billing_period_unit === "month") ? "Basic Member" : "Premier Member";
                            tables.push(tableItem);
                        });
                        this.json_data = tables;
                        this.filename = "All_Members";
                        this.currentTitle = "All Members";
                        this.number = tables.length !== 0 ? tables.length : "No data";
                        break;
                    default:
                        break;
                }
            } else {
                switch (event.target.value) {
                    case "current_basic":
                        this.chargebeeData.forEach(item => {
                            if (item.subscription.billing_period_unit !== "year") {
                                if((item.subscription.status === "active" || item.subscription.status === "non_renewing") && (moment(item.subscription.created_at * 1000).toDate() > moment(this.date).toDate())) {
                                    let tableItem = {};
                                    tableItem["id"] = item.customer.email;
                                    tableItem["status"] = item.subscription.status.toUpperCase();
                                    tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                    tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                    tableItem["recurring"] = "Basic Member";
                                    tables.push(tableItem);
                                }
                            }
                        });
                        this.json_data = tables;
                        this.filename = "Current_Basic";
                        this.currentTitle = "Current Basic Members";
                        this.number = tables.length !== 0 ? tables.length : "No data";
                        break;
                    case "current_premier":
                        this.chargebeeData.forEach(item => {
                            if (item.subscription.billing_period_unit === "year") {
                                if((item.subscription.status === "active" || item.subscription.status === "non_renewing") && (moment(item.subscription.created_at * 1000).toDate() > moment(this.date).toDate())) {
                                    let tableItem = {};
                                    tableItem["id"] = item.customer.email;
                                    tableItem["status"] = item.subscription.status.toUpperCase();
                                    tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                    tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                    tableItem["recurring"] = "Premier Member";
                                    tables.push(tableItem);
                                }
                            }
                        });
                        this.json_data = tables;
                        this.filename = "Current_Premier";
                        this.currentTitle = "Current Premier Members";
                        this.number = tables.length !== 0 ? tables.length : "No data";
                        break;
                    case "new_monthly_basic_purchase":
                        this.chargebeeData.forEach(item => {
                            if (item.subscription.billing_period_unit !== "year" && (now.getMonth() === moment(item.subscription.created_at * 1000).toDate().getMonth() && now.getYear() === moment(item.subscription.created_at * 1000).toDate().getYear()) && (moment(item.subscription.created_at * 1000).toDate() > moment(this.date).toDate())) {
                                let tableItem = {};
                                tableItem["id"] = item.customer.email;
                                tableItem["status"] = item.subscription.status.toUpperCase();
                                tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                tableItem["recurring"] = "Basic Member";
                                tables.push(tableItem);
                            }
                        });
                        this.json_data = tables;
                        this.filename = "New_Monthly_Basic_Purchase";
                        this.currentTitle = "New Monthly Basic Purchase Members";
                        this.number = tables.length !== 0 ? tables.length : "No data";
                        break;
                    case "monthly_basic_cancelled":
                        this.chargebeeData.forEach(item => {
                            if (item.subscription.billing_period_unit !== "year" && item.subscription.status === "cancelled" && item.subscription.billing_period === 1 && (moment(item.subscription.created_at * 1000).toDate() > moment(this.date).toDate())) {
                                let tableItem = {};
                                tableItem["id"] = item.customer.email;
                                tableItem["status"] = item.subscription.status.toUpperCase();
                                tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                tableItem["recurring"] = "Basic Member";
                                tables.push(tableItem);
                            }
                        });
                        this.json_data = tables;
                        this.filename = "Monthly_Basic_Cancelled";
                        this.currentTitle = "Monthly Basic Cancellations";
                        this.number = tables.length !== 0 ? tables.length : "No data";
                        break;
                    case "monthly_premier_purchase":
                        this.chargebeeData.forEach(item => {
                            if (item.subscription.billing_period_unit === "year" && item.subscription.status === "cancelled" && item.subscription.billing_period === 1 && (moment(item.subscription.created_at * 1000).toDate() > moment(this.date).toDate())) {
                                let tableItem = {};
                                tableItem["id"] = item.customer.email;
                                tableItem["status"] = item.subscription.status.toUpperCase();
                                tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                tableItem["recurring"] = "Premier Member";
                                tables.push(tableItem);
                            }
                        });
                        this.json_data = tables;
                        this.filename = "Monthly_Premier_Purchase";
                        this.currentTitle = "Monthly Premier Purchase Members";
                        this.number = tables.length !== 0 ? tables.length : "No data";
                        break;
                    case "premier_schedule":
                        this.chargebeeData.forEach(item => {
                            if (item.subscription.billing_period_unit === "year" && item.subscription.billing_period === 1 && (moment(item.subscription.created_at * 1000).toDate() > moment(this.date).toDate())) {
                                let tableItem = {};
                                tableItem["id"] = item.customer.email;
                                tableItem["status"] = item.subscription.status.toUpperCase();
                                tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                tableItem["recurring"] = "Premier Member";
                                tables.push(tableItem);
                            }
                        });
                        this.json_data = tables;
                        this.filename = "Premier_Schedule";
                        this.currentTitle = "Premier Schedule Members";
                        this.number = tables.length !== 0 ? tables.length : "No data";
                        break;
                    case "":
                        this.chargebeeData.forEach(item => {
                            if (moment(item.subscription.created_at * 1000).toDate() > moment(this.date).toDate()) {
                                let tableItem = {};
                                tableItem["id"] = item.customer.email;
                                tableItem["status"] = item.subscription.status.toUpperCase();
                                tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                tableItem["recurring"] = (item.subscription.billing_period_unit === "month") ? "Basic Member" : "Premier Member";
                                tables.push(tableItem);
                            }
                        });
                        this.json_data = tables;
                        this.filename = "All_Members";
                        this.currentTitle = "All Members";
                        this.number = tables.length !== 0 ? tables.length : "No data";
                        break;
                    default:
                        break;
                }
            }
        }
    },
    changeDate: function (date) {
        // search by date
        let tables = [];
        if (this.activeFlag) {
            if (this.chargebeeFlag) {
                if (date !== null) {
                    switch (this.$refs.chargebee.value) {
                        case "current_basic":
                            this.chargebeeData.forEach(item => {
                                if (item.subscription.billing_period_unit !== "year") {
                                    if((item.subscription.status === "active" || item.subscription.status === "non_renewing") && (moment(item.subscription.created_at * 1000).toDate() > moment(date).toDate())) {
                                        let tableItem = {};
                                        tableItem["id"] = item.customer.email;
                                        tableItem["status"] = item.subscription.status.toUpperCase();
                                        tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                        tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                        tableItem["recurring"] = "Basic Member";
                                        tables.push(tableItem);
                                    }
                                }
                            });
                            this.json_data = tables;
                            this.filename = "Current_Basic";
                            this.currentTitle = "Current Basic Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "current_premier":
                            this.chargebeeData.forEach(item => {
                                if (item.subscription.billing_period_unit === "year") {
                                    if((item.subscription.status === "active" || item.subscription.status === "non_renewing") && (moment(item.subscription.created_at * 1000).toDate() > moment(date).toDate())) {
                                        let tableItem = {};
                                        tableItem["id"] = item.customer.email;
                                        tableItem["status"] = item.subscription.status.toUpperCase();
                                        tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                        tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                        tableItem["recurring"] = "Premier Member";
                                        tables.push(tableItem);
                                    }
                                }
                            });
                            this.json_data = tables;
                            this.filename = "Current_Premier";
                            this.currentTitle = "Current Premier Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "new_monthly_basic_purchase":
                            this.chargebeeData.forEach(item => {
                                if (item.subscription.billing_period_unit !== "year" && (now.getMonth() === moment(item.subscription.created_at * 1000).toDate().getMonth() && now.getYear() === moment(item.subscription.created_at * 1000).toDate().getYear()) && (moment(item.subscription.created_at * 1000).toDate() > moment(date).toDate())) {
                                    let tableItem = {};
                                    tableItem["id"] = item.customer.email;
                                    tableItem["status"] = item.subscription.status.toUpperCase();
                                    tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                    tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                    tableItem["recurring"] = "Basic Member";
                                    tables.push(tableItem);
                                }
                            });
                            this.json_data = tables;
                            this.filename = "New_Monthly_Basic_Purchase";
                            this.currentTitle = "New Monthly Basic Purchase Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "monthly_basic_cancelled":
                            this.chargebeeData.forEach(item => {
                                if (item.subscription.billing_period_unit !== "year" && item.subscription.status === "cancelled" && item.subscription.billing_period === 1 && (moment(item.subscription.created_at * 1000).toDate() > moment(date).toDate())) {
                                    let tableItem = {};
                                    tableItem["id"] = item.customer.email;
                                    tableItem["status"] = item.subscription.status.toUpperCase();
                                    tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                    tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                    tableItem["recurring"] = "Basic Member";
                                    tables.push(tableItem);
                                }
                            });
                            this.json_data = tables;
                            this.filename = "Monthly_Basic_Cancelled";
                            this.currentTitle = "Monthly Basic Cancellations";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "monthly_premier_purchase":
                            this.chargebeeData.forEach(item => {
                                if (item.subscription.billing_period_unit === "year" && item.subscription.status === "cancelled" && item.subscription.billing_period === 1 && (moment(item.subscription.created_at * 1000).toDate() > moment(date).toDate())) {
                                    let tableItem = {};
                                    tableItem["id"] = item.customer.email;
                                    tableItem["status"] = item.subscription.status.toUpperCase();
                                    tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                    tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                    tableItem["recurring"] = "Premier Member";
                                    tables.push(tableItem);
                                }
                            });
                            this.json_data = tables;
                            this.filename = "Monthly_Premier_Purchase";
                            this.currentTitle = "Monthly Premier Purchase Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "premier_schedule":
                            this.chargebeeData.forEach(item => {
                                if (item.subscription.billing_period_unit === "year" && item.subscription.billing_period === 1 && (moment(item.subscription.created_at * 1000).toDate() > moment(date).toDate())) {
                                    let tableItem = {};
                                    tableItem["id"] = item.customer.email;
                                    tableItem["status"] = item.subscription.status.toUpperCase();
                                    tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                    tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                    tableItem["recurring"] = "Premier Member";
                                    tables.push(tableItem);
                                }
                            });
                            this.json_data = tables;
                            this.filename = "Premier_Schedule";
                            this.currentTitle = "Premier Schedule Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "":
                            this.chargebeeData.forEach(item => {
                                if (moment(item.subscription.created_at * 1000).toDate() > moment(date).toDate()) {
                                    let tableItem = {};
                                    tableItem["id"] = item.customer.email;
                                    tableItem["status"] = item.subscription.status.toUpperCase();
                                    tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                    tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                    tableItem["recurring"] = (item.subscription.billing_period_unit === "month") ? "Basic Member" : "Premier Member";
                                    tables.push(tableItem);
                                }
                            });
                            this.json_data = tables;
                            this.filename = "All_Members";
                            this.currentTitle = "All Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        default:
                            break;
                    }
                } else {
                    switch (this.$refs.chargebee.value) {
                        case "current_basic":
                            this.chargebeeData.forEach(item => {
                                if (item.subscription.billing_period_unit !== "year") {
                                    if(item.subscription.status === "active" || item.subscription.status === "non_renewing") {
                                        let tableItem = {};
                                        tableItem["id"] = item.customer.email;
                                        tableItem["status"] = item.subscription.status.toUpperCase();
                                        tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                        tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                        tableItem["recurring"] = "Basic Member";
                                        tables.push(tableItem);
                                    }
                                }
                            });
                            this.json_data = tables;
                            this.filename = "Current_Basic";
                            this.currentTitle = "Current Basic Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "current_premier":
                            this.chargebeeData.forEach(item => {
                                if (item.subscription.billing_period_unit === "year") {
                                    if(item.subscription.status === "active" || item.subscription.status === "non_renewing") {
                                        let tableItem = {};
                                        tableItem["id"] = item.customer.email;
                                        tableItem["status"] = item.subscription.status.toUpperCase();
                                        tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                        tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                        tableItem["recurring"] = "Premier Member";
                                        tables.push(tableItem);
                                    }
                                }
                            });
                            this.json_data = tables;
                            this.filename = "Current_Premier";
                            this.currentTitle = "Current Premier Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "new_monthly_basic_purchase":
                            this.chargebeeData.forEach(item => {
                                if (item.subscription.billing_period_unit !== "year" && (now.getMonth() === moment(item.subscription.created_at * 1000).toDate().getMonth() && now.getYear() === moment(item.subscription.created_at * 1000).toDate().getYear())) {
                                    let tableItem = {};
                                    tableItem["id"] = item.customer.email;
                                    tableItem["status"] = item.subscription.status.toUpperCase();
                                    tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                    tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                    tableItem["recurring"] = "Basic Member";
                                    tables.push(tableItem);
                                }
                            });
                            this.json_data = tables;
                            this.filename = "New_Monthly_Basic_Purchase";
                            this.currentTitle = "New Monthly Basic Purchase Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "monthly_basic_cancelled":
                            this.chargebeeData.forEach(item => {
                                if (item.subscription.billing_period_unit !== "year" && item.subscription.status === "cancelled" && item.subscription.billing_period === 1) {
                                    let tableItem = {};
                                    tableItem["id"] = item.customer.email;
                                    tableItem["status"] = item.subscription.status.toUpperCase();
                                    tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                    tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                    tableItem["recurring"] = "Basic Member";
                                    tables.push(tableItem);
                                }
                            });
                            this.json_data = tables;
                            this.filename = "Monthly_Basic_Cancelled";
                            this.currentTitle = "Monthly Basic Cancellations";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "monthly_premier_purchase":
                            this.chargebeeData.forEach(item => {
                                if (item.subscription.billing_period_unit === "year" && item.subscription.status === "cancelled" && item.subscription.billing_period === 1) {
                                    let tableItem = {};
                                    tableItem["id"] = item.customer.email;
                                    tableItem["status"] = item.subscription.status.toUpperCase();
                                    tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                    tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                    tableItem["recurring"] = "Premier Member";
                                    tables.push(tableItem);
                                }
                            });
                            this.json_data = tables;
                            this.filename = "Monthly_Premier_Purchase";
                            this.currentTitle = "Monthly Premier Purchase Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "premier_schedule":
                            this.chargebeeData.forEach(item => {
                                if (item.subscription.billing_period_unit === "year" && item.subscription.billing_period === 1) {
                                    let tableItem = {};
                                    tableItem["id"] = item.customer.email;
                                    tableItem["status"] = item.subscription.status.toUpperCase();
                                    tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                    tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                    tableItem["recurring"] = "Premier Member";
                                    tables.push(tableItem);
                                }
                            });
                            this.json_data = tables;
                            this.filename = "Premier_Schedule";
                            this.currentTitle = "Premier Schedule Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "":
                            this.chargebeeData.forEach(item => {
                                let tableItem = {};
                                tableItem["id"] = item.customer.email;
                                tableItem["status"] = item.subscription.status.toUpperCase();
                                tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                tableItem["recurring"] = (item.subscription.billing_period_unit === "month") ? "Basic Member" : "Premier Member";
                                tables.push(tableItem);
                            });
                            this.json_data = tables;
                            this.filename = "All_Members";
                            this.currentTitle = "All Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        default:
                            break;
                    }
                }
            } else {
                if (date === null) {
                    switch (this.$refs.activecampaign.value) {
                        case "current_group":
                            this.activecampaignData.forEach(item => {
                                item.tags.forEach(tag => {
                                    if (tag === "MemberType-Kitces-Report-Group-Member") {
                                        let tableItem = {};
                                        tableItem["name"] = item.full_name;
                                        tableItem["email"] = item.email;
                                        tableItem["phone"] = item.phone;
                                        tableItem["account"] = item.account;
                                        tableItem["date_created"] = moment(item.created_at).add(8, "hours").format("MM/DD/YYYY hh:mm a");
                                        tables.push(tableItem);
                                    }
                                });
                            });
                            this.json_data = tables;
                            this.filename = "Current_Group";
                            this.currentTitle = "Current Group Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "current_xypn":
                            this.activecampaignData.forEach(item => {
                                item.tags.forEach(tag => {
                                    if (tag === "MemberType-XYPN-Member-Subscription") {
                                        let tableItem = {};
                                        tableItem["name"] = item.full_name;
                                        tableItem["email"] = item.email;
                                        tableItem["phone"] = item.phone;
                                        tableItem["account"] = item.account;
                                        tableItem["date_created"] = moment(item.created_at).add(8, "hours").format("MM/DD/YYYY hh:mm a");
                                        tables.push(tableItem);
                                    }
                                });
                            });
                            this.json_data = tables;
                            this.filename = "Current_XYPN";
                            this.currentTitle = "Current XYPN Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "current_complimentary":
                            this.activecampaignData.forEach(item => {
                                item.tags.forEach(tag => {
                                    if (tag === "MemberType-The-Kitces-Report-Complimentary") {
                                        let tableItem = {};
                                        tableItem["name"] = item.full_name;
                                        tableItem["email"] = item.email;
                                        tableItem["phone"] = item.phone;
                                        tableItem["account"] = item.account;
                                        tableItem["date_created"] = moment(item.created_at).add(8, "hours").format("MM/DD/YYYY hh:mm a");
                                        tables.push(tableItem);
                                    }
                                });
                            });
                            this.json_data = tables;
                            this.filename = "Current_Complimentary";
                            this.currentTitle = "Current Complimentary Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "premier_yesterday":
                            this.activecampaignData.forEach(item => {
                                if (item.tags.indexOf("Reports-MemberRenewal-" + months[now.getMonth()] + "-2020-Expiration") !== -1 && item.tags.indexOf("Reports-MemberRenewal-" + months[now.getMonth()] + "-2021-Expiration")) {
                                    let tableItem = {};
                                    tableItem["name"] = item.full_name;
                                    tableItem["email"] = item.email;
                                    tableItem["phone"] = item.phone;
                                    tableItem["account"] = item.account;
                                    tableItem["date_created"] = moment(item.created_at).add(8, "hours").format("MM/DD/YYYY hh:mm a");
                                    tables.push(tableItem);
                                }
                            });
                            this.json_data = tables;
                            this.filename = "Premier_Yesterday";
                            this.currentTitle = "Premier Auto-Renewed Month to Yesterday";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "":
                            this.activecampaignData.forEach(item => {
                                let tableItem = {};
                                tableItem["name"] = item.full_name;
                                tableItem["email"] = item.email;
                                tableItem["phone"] = item.phone;
                                tableItem["account"] = item.account;
                                tableItem["date_created"] = moment(item.created_at).add(8, "hours").format("MM/DD/YYYY hh:mm a");
                                tables.push(tableItem);
                            });
                            this.json_data = tables;
                            this.filename = "All_Members";
                            this.currentTitle = "All ActiveCampaign Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        default:
                            break;
                    }
                } else {
                    switch (this.$refs.activecampaign.value) {
                        case "current_group":
                            this.activecampaignData.forEach(item => {
                                if (moment(item.created_at).toDate() > moment(date).toDate()) {
                                    item.tags.forEach(tag => {
                                        if (tag === "MemberType-Kitces-Report-Group-Member") {
                                            let tableItem = {};
                                            tableItem["name"] = item.full_name;
                                            tableItem["email"] = item.email;
                                            tableItem["phone"] = item.phone;
                                            tableItem["account"] = item.account;
                                            tableItem["date_created"] = moment(item.created_at).add(8, "hours").format("MM/DD/YYYY hh:mm a");
                                            tables.push(tableItem);
                                        }
                                    });
                                }
                            });
                            this.json_data = tables;
                            this.filename = "Current_Group";
                            this.currentTitle = "Current Group Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "current_xypn":
                            this.activecampaignData.forEach(item => {
                                if (moment(item.created_at).toDate() > moment(date).toDate()) {
                                    item.tags.forEach(tag => {
                                        if (tag === "MemberType-XYPN-Member-Subscription") {
                                            let tableItem = {};
                                            tableItem["name"] = item.full_name;
                                            tableItem["email"] = item.email;
                                            tableItem["phone"] = item.phone;
                                            tableItem["account"] = item.account;
                                            tableItem["date_created"] = moment(item.created_at).add(8, "hours").format("MM/DD/YYYY hh:mm a");
                                            tables.push(tableItem);
                                        }
                                    });
                                }
                            });
                            this.json_data = tables;
                            this.filename = "Current_XYPN";
                            this.currentTitle = "Current XYPN Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "current_complimentary":
                            this.activecampaignData.forEach(item => {
                                if (moment(item.created_at).toDate() > moment(date).toDate()) {
                                    item.tags.forEach(tag => {
                                        if (tag === "MemberType-The-Kitces-Report-Complimentary") {
                                            let tableItem = {};
                                            tableItem["name"] = item.full_name;
                                            tableItem["email"] = item.email;
                                            tableItem["phone"] = item.phone;
                                            tableItem["account"] = item.account;
                                            tableItem["date_created"] = moment(item.created_at).add(8, "hours").format("MM/DD/YYYY hh:mm a");
                                            tables.push(tableItem);
                                        }
                                    })
                                }
                            });
                            this.json_data = tables;
                            this.filename = "Current_Complimentary";
                            this.currentTitle = "Current Complimentary Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "premier_yesterday":
                            this.activecampaignData.forEach(item => {
                                if (moment(item.created_at).toDate() > moment(this.date).toDate()) {
                                    if (item.tags.indexOf("Reports-MemberRenewal-" + months[now.getMonth()] + "-2020-Expiration") !== -1 && item.tags.indexOf("Reports-MemberRenewal-" + months[now.getMonth()] + "-2021-Expiration")) {
                                        let tableItem = {};
                                        tableItem["name"] = item.full_name;
                                        tableItem["email"] = item.email;
                                        tableItem["phone"] = item.phone;
                                        tableItem["account"] = item.account;
                                        tableItem["date_created"] = moment(item.created_at).add(8, "hours").format("MM/DD/YYYY hh:mm a");
                                        tables.push(tableItem);
                                    }
                                }
                            });
                            this.json_data = tables;
                            this.filename = "Premier_Yesterday";
                            this.currentTitle = "Premier Auto-Renewed Month to Yesterday";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "":
                            this.activecampaignData.forEach(item => {
                                if (moment(item.created_at).toDate() > moment(date).toDate()) {
                                    let tableItem = {};
                                    tableItem["name"] = item.full_name;
                                    tableItem["email"] = item.email;
                                    tableItem["phone"] = item.phone;
                                    tableItem["account"] = item.account;
                                    tableItem["date_created"] = moment(item.created_at).add(8, "hours").format("MM/DD/YYYY hh:mm a");
                                    tables.push(tableItem);
                                }
                            });
                            this.json_data = tables;
                            this.filename = "All_Members";
                            this.currentTitle = "All ActiveCampaign Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        default:
                            break;
                    }
                }
            }
        }
    },
    onChangeActive: function (event) {
        let tables = [];
        // search by select
        if (this.activeFlag) {
            if (this.date === null) {
                switch (event.target.value) {
                    case "current_group":
                        this.activecampaignData.forEach(item => {
                            item.tags.forEach(tag => {
                                if (tag === "MemberType-Kitces-Report-Group-Member") {
                                    let tableItem = {};
                                    tableItem["name"] = item.full_name;
                                    tableItem["email"] = item.email;
                                    tableItem["phone"] = item.phone;
                                    tableItem["account"] = item.account;
                                    tableItem["date_created"] = moment(item.created_at).add(8, "hours").format("MM/DD/YYYY hh:mm a");
                                    tables.push(tableItem);
                                }
                            });
                        });
                        this.json_data = tables;
                        this.filename = "Current_Group";
                        this.currentTitle = "Current Group Members";
                        this.number = tables.length !== 0 ? tables.length : "No data";
                        break;
                    case "current_xypn":
                        this.activecampaignData.forEach(item => {
                            item.tags.forEach(tag => {
                                if (tag === "MemberType-XYPN-Member-Subscription") {
                                    let tableItem = {};
                                    tableItem["name"] = item.full_name;
                                    tableItem["email"] = item.email;
                                    tableItem["phone"] = item.phone;
                                    tableItem["account"] = item.account;
                                    tableItem["date_created"] = moment(item.created_at).add(8, "hours").format("MM/DD/YYYY hh:mm a");
                                    tables.push(tableItem);
                                }
                            });
                        });
                        this.json_data = tables;
                        this.filename = "Current_XYPN";
                        this.currentTitle = "Current XYPN Members";
                        this.number = tables.length !== 0 ? tables.length : "No data";
                        break;
                    case "current_complimentary":
                        this.activecampaignData.forEach(item => {
                            item.tags.forEach(tag => {
                                if (tag === "MemberType-The-Kitces-Report-Complimentary") {
                                    let tableItem = {};
                                    tableItem["name"] = item.full_name;
                                    tableItem["email"] = item.email;
                                    tableItem["phone"] = item.phone;
                                    tableItem["account"] = item.account;
                                    tableItem["date_created"] = moment(item.created_at).add(8, "hours").format("MM/DD/YYYY hh:mm a");
                                    tables.push(tableItem);
                                }
                            });
                        });
                        this.json_data = tables;
                        this.filename = "Current_Complimentary";
                        this.currentTitle = "Current Complimentary Members";
                        this.number = tables.length !== 0 ? tables.length : "No data";
                        break;
                    case "premier_yesterday":
                        this.activecampaignData.forEach(item => {
                            if (item.tags.indexOf("Reports-MemberRenewal-" + months[now.getMonth()] + "-2020-Expiration") !== -1 && item.tags.indexOf("Reports-MemberRenewal-" + months[now.getMonth()] + "-2021-Expiration")) {
                                let tableItem = {};
                                tableItem["name"] = item.full_name;
                                tableItem["email"] = item.email;
                                tableItem["phone"] = item.phone;
                                tableItem["account"] = item.account;
                                tableItem["date_created"] = moment(item.created_at).add(8, "hours").format("MM/DD/YYYY hh:mm a");
                                tables.push(tableItem);
                            }
                        });
                        this.json_data = tables;
                        this.filename = "Premier_Yesterday";
                        this.currentTitle = "Premier Auto-Renewed Month to Yesterday";
                        this.number = tables.length !== 0 ? tables.length : "No data";
                        break;
                    case "":
                        this.activecampaignData.forEach(item => {
                            let tableItem = {};
                            tableItem["name"] = item.full_name;
                            tableItem["email"] = item.email;
                            tableItem["phone"] = item.phone;
                            tableItem["account"] = item.account;
                            tableItem["date_created"] = moment(item.created_at).add(8, "hours").format("MM/DD/YYYY hh:mm a");
                            tables.push(tableItem);
                        });
                        this.json_data = tables;
                        this.filename = "All_Members";
                        this.currentTitle = "All ActiveCampaign Members";
                        this.number = tables.length !== 0 ? tables.length : "No data";
                        break;
                    default:
                        break;
                }
            } else {
                switch (event.target.value) {
                    case "current_group":
                        this.activecampaignData.forEach(item => {
                            if (moment(item.created_at).toDate() > moment(this.date).toDate()) {
                                item.tags.forEach(tag => {
                                    if (tag === "MemberType-Kitces-Report-Group-Member") {
                                        let tableItem = {};
                                        tableItem["name"] = item.full_name;
                                        tableItem["email"] = item.email;
                                        tableItem["phone"] = item.phone;
                                        tableItem["account"] = item.account;
                                        tableItem["date_created"] = moment(item.created_at).add(8, "hours").format("MM/DD/YYYY hh:mm a");
                                        tables.push(tableItem);
                                    }
                                });
                            }
                        });
                        this.json_data = tables;
                        this.filename = "Current_Group";
                        this.currentTitle = "Current Group Members";
                        this.number = tables.length !== 0 ? tables.length : "No data";
                        break;
                    case "current_xypn":
                        this.activecampaignData.forEach(item => {
                            if (moment(item.created_at).toDate() > moment(this.date).toDate()) {
                                item.tags.forEach(tag => {
                                    if (tag === "MemberType-XYPN-Member-Subscription") {
                                        let tableItem = {};
                                        tableItem["name"] = item.full_name;
                                        tableItem["email"] = item.email;
                                        tableItem["phone"] = item.phone;
                                        tableItem["account"] = item.account;
                                        tableItem["date_created"] = moment(item.created_at).add(8, "hours").format("MM/DD/YYYY hh:mm a");
                                        tables.push(tableItem);
                                    }
                                });
                            }
                        });
                        this.json_data = tables;
                        this.filename = "Current_XYPN";
                        this.currentTitle = "Current XYPN Members";
                        this.number = tables.length !== 0 ? tables.length : "No data";
                        break;
                    case "current_complimentary":
                        this.activecampaignData.forEach(item => {
                            if (moment(item.created_at).toDate() > moment(this.date).toDate()) {
                                item.tags.forEach(tag => {
                                    if (tag === "MemberType-The-Kitces-Report-Complimentary") {
                                        let tableItem = {};
                                        tableItem["name"] = item.full_name;
                                        tableItem["email"] = item.email;
                                        tableItem["phone"] = item.phone;
                                        tableItem["account"] = item.account;
                                        tableItem["date_created"] = moment(item.created_at).add(8, "hours").format("MM/DD/YYYY hh:mm a");
                                        tables.push(tableItem);
                                    }
                                })
                            }
                        });
                        this.json_data = tables;
                        this.filename = "Current_Complimentary";
                        this.currentTitle = "Current Complimentary Members";
                        this.number = tables.length !== 0 ? tables.length : "No data";
                        break;
                    case "premier_yesterday":
                        this.activecampaignData.forEach(item => {
                            if (moment(item.created_at).toDate() > moment(this.date).toDate()) {
                                if (item.tags.indexOf("Reports-MemberRenewal-" + months[now.getMonth()] + "-2020-Expiration") !== -1 && item.tags.indexOf("Reports-MemberRenewal-" + months[now.getMonth()] + "-2021-Expiration")) {
                                    let tableItem = {};
                                    tableItem["name"] = item.full_name;
                                    tableItem["email"] = item.email;
                                    tableItem["phone"] = item.phone;
                                    tableItem["account"] = item.account;
                                    tableItem["date_created"] = moment(item.created_at).add(8, "hours").format("MM/DD/YYYY hh:mm a");
                                    tables.push(tableItem);
                                }
                            }
                        });
                        this.json_data = tables;
                        this.filename = "Premier_Yesterday";
                        this.currentTitle = "Premier Auto-Renewed Month to Yesterday";
                        this.number = tables.length !== 0 ? tables.length : "No data";
                        break;
                    case "":
                        this.activecampaignData.forEach(item => {
                            if (moment(item.created_at).toDate() > moment(this.date).toDate()) {
                                let tableItem = {};
                                tableItem["name"] = item.full_name;
                                tableItem["email"] = item.email;
                                tableItem["phone"] = item.phone;
                                tableItem["account"] = item.account;
                                tableItem["date_created"] = moment(item.created_at).add(8, "hours").format("MM/DD/YYYY hh:mm a");
                                tables.push(tableItem);
                            }
                        });
                        this.json_data = tables;
                        this.filename = "All_Members";
                        this.currentTitle = "All ActiveCampaign Members";
                        this.number = tables.length !== 0 ? tables.length : "No data";
                        break;
                    default:
                        break;
                }
            }
        }
    },
    changeStyle: function (style) {
        if (this.activeFlag) {
            if (style === 1) {
                this.chargebeeFlag = true;
                let tables = [];
                // search by select
                if (this.date === null) {
                    switch (this.$refs.chargebee.value) {
                        case "current_basic":
                            this.chargebeeData.forEach(item => {
                                if (item.subscription.billing_period_unit !== "year") {
                                    if(item.subscription.status === "active" || item.subscription.status === "non_renewing") {
                                        let tableItem = {};
                                        tableItem["id"] = item.customer.email;
                                        tableItem["status"] = item.subscription.status.toUpperCase();
                                        tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                        tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                        tableItem["recurring"] = "Basic Member";
                                        tables.push(tableItem);
                                    }
                                }
                            });
                            this.json_data = tables;
                            this.filename = "Current_Basic";
                            this.currentTitle = "Current Basic Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "current_premier":
                            this.chargebeeData.forEach(item => {
                                if (item.subscription.billing_period_unit === "year") {
                                    if(item.subscription.status === "active" || item.subscription.status === "non_renewing") {
                                        let tableItem = {};
                                        tableItem["id"] = item.customer.email;
                                        tableItem["status"] = item.subscription.status.toUpperCase();
                                        tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                        tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                        tableItem["recurring"] = "Premier Member";
                                        tables.push(tableItem);
                                    }
                                }
                            });
                            this.json_data = tables;
                            this.filename = "Current_Premier";
                            this.currentTitle = "Current Premier Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "new_monthly_basic_purchase":
                            this.chargebeeData.forEach(item => {
                                if (item.subscription.billing_period_unit !== "year" && (now.getMonth() === moment(item.subscription.created_at * 1000).toDate().getMonth() && now.getYear() === moment(item.subscription.created_at * 1000).toDate().getYear())) {
                                    let tableItem = {};
                                    tableItem["id"] = item.customer.email;
                                    tableItem["status"] = item.subscription.status.toUpperCase();
                                    tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                    tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                    tableItem["recurring"] = "Basic Member";
                                    tables.push(tableItem);
                                }
                            });
                            this.json_data = tables;
                            this.filename = "New_Monthly_Basic_Purchase";
                            this.currentTitle = "New Monthly Basic Purchase Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "monthly_basic_cancelled":
                            this.chargebeeData.forEach(item => {
                                if (item.subscription.billing_period_unit !== "year" && item.subscription.status === "cancelled" && item.subscription.billing_period === 1) {
                                    let tableItem = {};
                                    tableItem["id"] = item.customer.email;
                                    tableItem["status"] = item.subscription.status.toUpperCase();
                                    tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                    tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                    tableItem["recurring"] = "Basic Member";
                                    tables.push(tableItem);
                                }
                            });
                            this.json_data = tables;
                            this.filename = "Monthly_Basic_Cancelled";
                            this.currentTitle = "Monthly Basic Cancellations";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "monthly_premier_purchase":
                            this.chargebeeData.forEach(item => {
                                if (item.subscription.billing_period_unit === "year" && item.subscription.status === "cancelled" && item.subscription.billing_period === 1) {
                                    let tableItem = {};
                                    tableItem["id"] = item.customer.email;
                                    tableItem["status"] = item.subscription.status.toUpperCase();
                                    tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                    tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                    tableItem["recurring"] = "Premier Member";
                                    tables.push(tableItem);
                                }
                            });
                            this.json_data = tables;
                            this.filename = "Monthly_Premier_Purchase";
                            this.currentTitle = "Monthly Premier Purchase Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "premier_schedule":
                            this.chargebeeData.forEach(item => {
                                if (item.subscription.billing_period_unit === "year" && item.subscription.billing_period === 1) {
                                    let tableItem = {};
                                    tableItem["id"] = item.customer.email;
                                    tableItem["status"] = item.subscription.status.toUpperCase();
                                    tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                    tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                    tableItem["recurring"] = "Premier Member";
                                    tables.push(tableItem);
                                }
                            });
                            this.json_data = tables;
                            this.filename = "Premier_Schedule";
                            this.currentTitle = "Premier Schedule Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "":
                            this.chargebeeData.forEach(item => {
                                let tableItem = {};
                                tableItem["id"] = item.customer.email;
                                tableItem["status"] = item.subscription.status.toUpperCase();
                                tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                tableItem["recurring"] = (item.subscription.billing_period_unit === "month") ? "Basic Member" : "Premier Member";
                                tables.push(tableItem);
                            });
                            this.json_data = tables;
                            this.filename = "All_Members";
                            this.currentTitle = "All Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        default:
                            break;
                    }
                } else {
                    switch (this.$refs.chargebee.value) {
                        case "current_basic":
                            this.chargebeeData.forEach(item => {
                                if (item.subscription.billing_period_unit !== "year") {
                                    if((item.subscription.status === "active" || item.subscription.status === "non_renewing") && (moment(item.subscription.created_at * 1000).toDate() > moment(this.date).toDate())) {
                                        let tableItem = {};
                                        tableItem["id"] = item.customer.email;
                                        tableItem["status"] = item.subscription.status.toUpperCase();
                                        tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                        tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                        tableItem["recurring"] = "Basic Member";
                                        tables.push(tableItem);
                                    }
                                }
                            });
                            this.json_data = tables;
                            this.filename = "Current_Basic";
                            this.currentTitle = "Current Basic Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "current_premier":
                            this.chargebeeData.forEach(item => {
                                if (item.subscription.billing_period_unit === "year") {
                                    if((item.subscription.status === "active" || item.subscription.status === "non_renewing") && (moment(item.subscription.created_at * 1000).toDate() > moment(this.date).toDate())) {
                                        let tableItem = {};
                                        tableItem["id"] = item.customer.email;
                                        tableItem["status"] = item.subscription.status.toUpperCase();
                                        tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                        tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                        tableItem["recurring"] = "Premier Member";
                                        tables.push(tableItem);
                                    }
                                }
                            });
                            this.json_data = tables;
                            this.filename = "Current_Premier";
                            this.currentTitle = "Current Premier Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "new_monthly_basic_purchase":
                            this.chargebeeData.forEach(item => {
                                if (item.subscription.billing_period_unit !== "year" && (now.getMonth() === moment(item.subscription.created_at * 1000).toDate().getMonth() && now.getYear() === moment(item.subscription.created_at * 1000).toDate().getYear()) && (moment(item.subscription.created_at * 1000).toDate() > moment(this.date).toDate())) {
                                    let tableItem = {};
                                    tableItem["id"] = item.customer.email;
                                    tableItem["status"] = item.subscription.status.toUpperCase();
                                    tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                    tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                    tableItem["recurring"] = "Basic Member";
                                    tables.push(tableItem);
                                }
                            });
                            this.json_data = tables;
                            this.filename = "New_Monthly_Basic_Purchase";
                            this.currentTitle = "New Monthly Basic Purchase Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "monthly_basic_cancelled":
                            this.chargebeeData.forEach(item => {
                                if (item.subscription.billing_period_unit !== "year" && item.subscription.status === "cancelled" && item.subscription.billing_period === 1 && (moment(item.subscription.created_at * 1000).toDate() > moment(this.date).toDate())) {
                                    let tableItem = {};
                                    tableItem["id"] = item.customer.email;
                                    tableItem["status"] = item.subscription.status.toUpperCase();
                                    tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                    tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                    tableItem["recurring"] = "Basic Member";
                                    tables.push(tableItem);
                                }
                            });
                            this.json_data = tables;
                            this.filename = "Monthly_Basic_Cancelled";
                            this.currentTitle = "Monthly Basic Cancellations";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "monthly_premier_purchase":
                            this.chargebeeData.forEach(item => {
                                if (item.subscription.billing_period_unit === "year" && item.subscription.status === "cancelled" && item.subscription.billing_period === 1 && (moment(item.subscription.created_at * 1000).toDate() > moment(this.date).toDate())) {
                                    let tableItem = {};
                                    tableItem["id"] = item.customer.email;
                                    tableItem["status"] = item.subscription.status.toUpperCase();
                                    tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                    tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                    tableItem["recurring"] = "Premier Member";
                                    tables.push(tableItem);
                                }
                            });
                            this.json_data = tables;
                            this.filename = "Monthly_Premier_Purchase";
                            this.currentTitle = "Monthly Premier Purchase Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "premier_schedule":
                            this.chargebeeData.forEach(item => {
                                if (item.subscription.billing_period_unit === "year" && item.subscription.billing_period === 1 && (moment(item.subscription.created_at * 1000).toDate() > moment(this.date).toDate())) {
                                    let tableItem = {};
                                    tableItem["id"] = item.customer.email;
                                    tableItem["status"] = item.subscription.status.toUpperCase();
                                    tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                    tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                    tableItem["recurring"] = "Premier Member";
                                    tables.push(tableItem);
                                }
                            });
                            this.json_data = tables;
                            this.filename = "Premier_Schedule";
                            this.currentTitle = "Premier Schedule Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "":
                            this.chargebeeData.forEach(item => {
                                if (moment(item.subscription.created_at * 1000).toDate() > moment(this.date).toDate()) {
                                    let tableItem = {};
                                    tableItem["id"] = item.customer.email;
                                    tableItem["status"] = item.subscription.status.toUpperCase();
                                    tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
                                    tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
                                    tableItem["recurring"] = (item.subscription.billing_period_unit === "month") ? "Basic Member" : "Premier Member";
                                    tables.push(tableItem);
                                }
                            });
                            this.json_data = tables;
                            this.filename = "All_Members";
                            this.currentTitle = "All Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        default:
                            break;
                    }
                }
                this.currentTitle = "All Members";
            } else {
                this.chargebeeFlag = false;
                let tables = [];
                // search by select
                if (this.date === null) {
                    switch (this.$refs.activecampaign.value) {
                        case "current_group":
                            this.activecampaignData.forEach(item => {
                                item.tags.forEach(tag => {
                                    if (tag === "MemberType-Kitces-Report-Group-Member") {
                                        let tableItem = {};
                                        tableItem["name"] = item.full_name;
                                        tableItem["email"] = item.email;
                                        tableItem["phone"] = item.phone;
                                        tableItem["account"] = item.account;
                                        tableItem["date_created"] = moment(item.created_at).add(8, "hours").format("MM/DD/YYYY hh:mm a");
                                        tables.push(tableItem);
                                    }
                                });
                            });
                            this.json_data = tables;
                            this.filename = "Current_Group";
                            this.currentTitle = "Current Group Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "current_xypn":
                            this.activecampaignData.forEach(item => {
                                item.tags.forEach(tag => {
                                    if (tag === "MemberType-XYPN-Member-Subscription") {
                                        let tableItem = {};
                                        tableItem["name"] = item.full_name;
                                        tableItem["email"] = item.email;
                                        tableItem["phone"] = item.phone;
                                        tableItem["account"] = item.account;
                                        tableItem["date_created"] = moment(item.created_at).add(8, "hours").format("MM/DD/YYYY hh:mm a");
                                        tables.push(tableItem);
                                    }
                                });
                            });
                            this.json_data = tables;
                            this.filename = "Current_XYPN";
                            this.currentTitle = "Current XYPN Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "current_complimentary":
                            this.activecampaignData.forEach(item => {
                                item.tags.forEach(tag => {
                                    if (tag === "MemberType-The-Kitces-Report-Complimentary") {
                                        let tableItem = {};
                                        tableItem["name"] = item.full_name;
                                        tableItem["email"] = item.email;
                                        tableItem["phone"] = item.phone;
                                        tableItem["account"] = item.account;
                                        tableItem["date_created"] = moment(item.created_at).add(8, "hours").format("MM/DD/YYYY hh:mm a");
                                        tables.push(tableItem);
                                    }
                                });
                            });
                            this.json_data = tables;
                            this.filename = "Current_Complimentary";
                            this.currentTitle = "Current Complimentary Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "premier_yesterday":
                            this.activecampaignData.forEach(item => {
                                if (item.tags.indexOf("Reports-MemberRenewal-" + months[now.getMonth()] + "-2020-Expiration") !== -1 && item.tags.indexOf("Reports-MemberRenewal-" + months[now.getMonth()] + "-2021-Expiration")) {
                                    let tableItem = {};
                                    tableItem["name"] = item.full_name;
                                    tableItem["email"] = item.email;
                                    tableItem["phone"] = item.phone;
                                    tableItem["account"] = item.account;
                                    tableItem["date_created"] = moment(item.created_at).add(8, "hours").format("MM/DD/YYYY hh:mm a");
                                    tables.push(tableItem);
                                }
                            });
                            this.json_data = tables;
                            this.filename = "Premier_Yesterday";
                            this.currentTitle = "Premier Auto-Renewed Month to Yesterday";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "":
                            this.activecampaignData.forEach(item => {
                                let tableItem = {};
                                tableItem["name"] = item.full_name;
                                tableItem["email"] = item.email;
                                tableItem["phone"] = item.phone;
                                tableItem["account"] = item.account;
                                tableItem["date_created"] = moment(item.created_at).add(8, "hours").format("MM/DD/YYYY hh:mm a");
                                tables.push(tableItem);
                            });
                            this.json_data = tables;
                            this.filename = "All_Members";
                            this.currentTitle = "All ActiveCampaign Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        default:
                            break;
                    }
                } else {
                    switch (this.$refs.activecampaign.value) {
                        case "current_group":
                            this.activecampaignData.forEach(item => {
                                if (moment(item.created_at).toDate() > moment(this.date).toDate()) {
                                    item.tags.forEach(tag => {
                                        if (tag === "MemberType-Kitces-Report-Group-Member") {
                                            let tableItem = {};
                                            tableItem["name"] = item.full_name;
                                            tableItem["email"] = item.email;
                                            tableItem["phone"] = item.phone;
                                            tableItem["account"] = item.account;
                                            tableItem["date_created"] = moment(item.created_at).add(8, "hours").format("MM/DD/YYYY hh:mm a");
                                            tables.push(tableItem);
                                        }
                                    });
                                }
                            });
                            this.json_data = tables;
                            this.filename = "Current_Group";
                            this.currentTitle = "Current Group Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "current_xypn":
                            this.activecampaignData.forEach(item => {
                                if (moment(item.created_at).toDate() > moment(this.date).toDate()) {
                                    item.tags.forEach(tag => {
                                        if (tag === "MemberType-XYPN-Member-Subscription") {
                                            let tableItem = {};
                                            tableItem["name"] = item.full_name;
                                            tableItem["email"] = item.email;
                                            tableItem["phone"] = item.phone;
                                            tableItem["account"] = item.account;
                                            tableItem["date_created"] = moment(item.created_at).add(8, "hours").format("MM/DD/YYYY hh:mm a");
                                            tables.push(tableItem);
                                        }
                                    });
                                }
                            });
                            this.json_data = tables;
                            this.filename = "Current_XYPN";
                            this.currentTitle = "Current XYPN Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "current_complimentary":
                            this.activecampaignData.forEach(item => {
                                if (moment(item.created_at).toDate() > moment(this.date).toDate()) {
                                    item.tags.forEach(tag => {
                                        if (tag === "MemberType-The-Kitces-Report-Complimentary") {
                                            let tableItem = {};
                                            tableItem["name"] = item.full_name;
                                            tableItem["email"] = item.email;
                                            tableItem["phone"] = item.phone;
                                            tableItem["account"] = item.account;
                                            tableItem["date_created"] = moment(item.created_at).add(8, "hours").format("MM/DD/YYYY hh:mm a");
                                            tables.push(tableItem);
                                        }
                                    })
                                }
                            });
                            this.json_data = tables;
                            this.filename = "Current_Complimentary";
                            this.currentTitle = "Current Complimentary Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "premier_yesterday":
                            this.activecampaignData.forEach(item => {
                                if (moment(item.created_at).toDate() > moment(this.date).toDate()) {
                                    if (item.tags.indexOf("Reports-MemberRenewal-" + months[now.getMonth()] + "-2020-Expiration") !== -1 && item.tags.indexOf("Reports-MemberRenewal-" + months[now.getMonth()] + "-2021-Expiration")) {
                                        let tableItem = {};
                                        tableItem["name"] = item.full_name;
                                        tableItem["email"] = item.email;
                                        tableItem["phone"] = item.phone;
                                        tableItem["account"] = item.account;
                                        tableItem["date_created"] = moment(item.created_at).add(8, "hours").format("MM/DD/YYYY hh:mm a");
                                        tables.push(tableItem);
                                    }
                                }
                            });
                            this.json_data = tables;
                            this.filename = "Premier_Yesterday";
                            this.currentTitle = "Premier Auto-Renewed Month to Yesterday";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        case "":
                            this.activecampaignData.forEach(item => {
                                if (moment(item.created_at).toDate() > moment(this.date).toDate()) {
                                    let tableItem = {};
                                    tableItem["name"] = item.full_name;
                                    tableItem["email"] = item.email;
                                    tableItem["phone"] = item.phone;
                                    tableItem["account"] = item.account;
                                    tableItem["date_created"] = moment(item.created_at).add(8, "hours").format("MM/DD/YYYY hh:mm a");
                                    tables.push(tableItem);
                                }
                            });
                            this.json_data = tables;
                            this.filename = "All_Members";
                            this.currentTitle = "All ActiveCampaign Members";
                            this.number = tables.length !== 0 ? tables.length : "No data";
                            break;
                        default:
                            break;
                    }
                }
            }
        }
    }
  },
  async mounted () {
    let totalData = await api.getChargebees();
    console.log('totalData', totalData);
    this.chargebeeFlag = true;
    this.activeFlag = true;
    
    this.chargebeeData = totalData.chargebee[0].chargebee;
    this.activecampaignData = totalData.activecampaign;
    // save data from chargebee
    let chargebeeTables = [];
    totalData.chargebee[0].chargebee.forEach(item => {
        let tableItem = {};
        tableItem["id"] = item.customer.email;

        // check basic or premier
        if (item.subscription.billing_period_unit === "year") {
            tableItem["recurring"] = "Premier Member";
        } else {
            tableItem["recurring"] = "Basic Member";
        }

        switch (item.subscription.status) {
            case "future":
                tableItem["status"] = "FUTURE";
                break;
            case "in_trial":
                tableItem["status"] = "IN TRIAL";
                break;
            case "active":
                tableItem["status"] = "ACTIVE";
                break;
            case "non_renewing":
                tableItem["status"] = "NON RENEWING";
                break;
            case "paused":
                tableItem["status"] = "PAUSED";
                break;
            case "cancelled":
                tableItem["status"] = "CANCELLED";
                break;
            default:
                break;
        }
        tableItem["created_at"] = moment(item.subscription.created_at * 1000).format("DD-MM-YYYY");
        tableItem["next_billing"] = moment(item.subscription.next_billing_at * 1000).format("DD-MM-YYYY");
        chargebeeTables.push(tableItem);
    });

    this.chargebee_data = chargebeeTables;
    this.json_data = chargebeeTables;
    this.loadingFlag = false;
    this.currentTitle = "All Members";
    this.number = chargebeeTables.length;

    // save activecampaign data
    let activecampaignTables = [];
    totalData.activecampaign.forEach(item => {
        let table = {};
        table["name"] = item.full_name;
        table["email"] = item.email;
        table["phone"] = item.phone;
        table["account"] = item.account;
        table["date_created"] = moment(item.created_at).add(8, "hours").format("MM/DD/YYYY hh:mm a");
        activecampaignTables.push(table);
    });

    this.active_data = activecampaignTables;
  }
}
</script>
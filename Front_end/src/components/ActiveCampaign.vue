<template>
    <div class="all-member-tbl" v-if="visibleFlag && dataFlag">
        <h2>All Member table</h2>
        <mdb-datatable-2
            v-model="active_tbl_data"
        />
    </div>
</template>

<script>
import { mdbDatatable2 } from 'mdbvue';

export default {
    name: "ActiveCampaign",
    components: {
        mdbDatatable2
    },
    data () {
        return {
            active_tbl_data: {
                columns: [
                    {
                      label: 'Full Name',
                      field: 'name',
                      sort: true
                    },
                    {
                      label: 'Email',
                      field: 'email',
                      sort: true
                    },
                    {
                      label: 'Phone',
                      field: 'phone',
                      sort: true
                    },
                    {
                      label: 'Account',
                      field: 'account',
                      sort: true
                    },
                    {
                      label: 'Date Created',
                      field: 'date_created',
                      sort: true
                    }
                ],
                rows: null
            },
            dataFlag: false
        }
    },
    props: {
        data: {
            type: Array,
            required: false,
            default: null
        },
        visibleFlag: {
            type: Boolean,
            required: false,
            default: false
        }
    },
    updated () {
        this.active_tbl_data.rows = this.data;
        this.dataFlag = true;
    }
};
</script>
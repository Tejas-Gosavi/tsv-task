<template>
	<div id="container">
        <div id="input-container">
            <input type="text" id="input-box" autofocus v-model="wallet_address" placeholder="Enter Wallet Address">
            <button class="btn" type="button" id="fetch-btn" @click="fetchDate">Fetch</button>
			<button class="btn" id="clear-btn" @click="wallet_address = ''; contracts = []">Clear</button>
        </div>
        <div id="table-container">
            <table id="table">
                <thead>
                    <tr>
                        <th>Name</th>
                        <th>Sales (30 days)</th>
                        <th>Avg. price (30 days)</th>
                        <th>Ext. URL</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="(contract, idx) in contracts" :key="idx">
                        <td>{{ contract.name }}</td>
                        <td>{{ contract.sales30Days }}</td>
                        <td>{{ contract.avgPrice30Days }}</td>
						<td v-if="contract.externalLink === '-' || contract.externalLink === null ">-</td>
                        <td v-else><a :href="contract.externalLink">{{ contract.name }}</a></td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<script>
import axios from "axios";

export default {
	name: "Main",
	data() {
		return {
			wallet_address: "",
			contracts: [],
		};
	},
	methods: {
		fetchDate() {

			// check if wallet is null/empty string or not then
			if (this.$data.wallet_address === "") {
					document.getElementById("input-box").style.borderColor = "red";
				} else {
					this.$data.contracts = [];
					document.getElementById("input-box").style.borderColor = "#42b983";

					// fetching collections for given wallet address with offset = 0 and limit = 50
					axios.get("https://api.opensea.io/api/v1/collections", {
							params: {
								asset_owner: this.$data.wallet_address,
								offset: 0,
								limit: 50,
							},
						}).then((res) => {

							// single contract object
							const contract = {
								name: "",
								sales30Days: 0,
								avgPrice30Days: 0,
								externalLink: "",
							};

							// for every collection from collections
							res.data.forEach((col) => {
								
								// if it's primary_asset_contracts array has length equal to 0
								if (col.primary_asset_contracts.length === 0) {
									
									// then setting - for name and link
									contract.name = "-";
									contract.externalLink = "-";
								} else {

									// else setting values given name and link
									contract.name = col.primary_asset_contracts[0].name;
									contract.externalLink =
										col.primary_asset_contracts[0].external_link;
								}

								// setting 30 days sales and 30 days avg. price
								contract.sales30Days = col.stats.thirty_day_sales;
								contract.avgPrice30Days =
									col.stats.thirty_day_average_price.toFixed(5);
								
								// pushing single contract object in contracts array
								// using spread operator
								this.$data.contracts.push({ ...contract });
							});

							// comporator function to sort contracts array by 30 days sales
							function compare(a, b) {
								if (a.sales30Days < b.sales30Days) {
									return -1;
								}
								if (a.sales30Days > b.sales30Days) {
									return 1;
								}
								return 0;
							}

							// sorting contracts array based on comporator function
							this.$data.contracts.sort(compare);
						});
			}
		},
	},
};
</script>

<style scoped>
.container {
	margin: auto;
}
#input-box {
	outline: none !important;
	font-size: 16px;
	border: 1px solid #42b983;
	padding: 8px 20px;
	margin-right: 5px;
}
.btn {
	border: none;
	color: white;
	padding: 8px 20px;
	text-align: center;
	display: inline-block;
	font-size: 16px;
	transition-duration: 0.4s;
}
.btn:hover {
	opacity: 0.8;
}
#fetch-btn {
	background-color: #42b983;
	margin-right: 5px;
}
#clear-btn {
	background-color: #008CBA;
}
#table-container {
	margin-top: 20px;
}
#table {
	margin: auto;
	padding: auto;
	border-collapse: separate;
	border-spacing: 0;
}
table, th, td {
	border: 1px solid;
}
th, td {
	padding: 10px;
}
a {
	color: #42b983;
	text-decoration: none;
}
</style>

<template>
	<view>
		<view style="display: flex;flex-direction: column;justify-content: center;align-items: center;">
			<view style="margin-top: 20px;margin-top: 20px;">
				<text style="font-size: 20px;">
					{{county}}
				</text>
				<van-icon name="location" size="20px" />
			</view>
			<text style="font-size: 100px;margin-top: 20px;">{{res.data.data.observe.degree}}℃</text>

			<text style="font-size: 30px;margin-top: 20px;">
				{{res.data.data.forecast_24h[1].max_degree}}℃/{{res.data.data.forecast_24h[1].min_degree}}℃
			</text>

			<text style="font-size: 30px;margin-top: 20px;">
				{{res.data.data.forecast_24h[1].day_weather}}
			</text>
		</view>
		<view style="margin-top: 20px;">
			<scroll-view scroll-x="true"  style="white-space:nowrap;display:flex">
				<!-- 
				 <view  style="background:black;width:100px;height:100px;display:inline-block"></view> -->
				<view v-for="(value,index) in res.data.data.forecast_1h"
					style="width:105px;height:110px;display:inline-block;text-align:center;">
					<view style="margin-top: 10px;">
						<text style="font-size: 20px;">{{value.update_time.slice(8,10)}}:00</text>
					</view>
					<view style="margin-top: 10px;">
						<text style="font-size: 20px;">{{value.weather}}</text>
					</view>
					<view style="margin-top: 10px;">
						<text style="font-size: 20px;">{{value.degree}}℃</text>
					</view>
				</view>
			</scroll-view>

		</view>
		<view style="margin-top: 20px;">
			<!-- 
				 <view  style="background:black;width:100px;height:100px;display:inline-block"></view> -->
			<view style="height:40px;margin-top: 10px;" v-for="(value,index) in res.data.data.forecast_24h">
				<van-col span="8" style="display:inline;text-align:center;">
					<text style="font-size: 20px;">
						{{value.time.slice(5,7)}}月{{value.time.slice(8,10)}}日
					</text>
				</van-col>
				<van-col span="8" style="display:inline;text-align:center;">
					<text style="font-size: 20px;">
						{{value.day_weather}}
					</text>
				</van-col>
				<van-col span="8" style="display:inline;text-align:center;">
					<text style="font-size: 20px;">
						{{value.max_degree}}℃/{{value.min_degree}}℃
					</text>
				</van-col>
			</view>

		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				latitude: "", //纬度
				longitude: "", //经度
				province: "", //省
				city: "", //城市
				county: "", //区
				res: ""
				// markers: [ //标记点用于在地图上显示标记的位置
				// 	{
				// 		id: 1,
				// 		latitude: "", //纬度
				// 		longitude: "", //经度

				// 		width: "30", //图标的宽
				// 		height: "30" //图标的高
				// 	}
				// ]
			}
		},

		onLoad() {
			let that = this;
			// var QQMapWX = require('../../static/qqmap-wx-jssdk.js');
			// var qqmapsdk = new QQMapWX({
			// 	key: 'QHCBZ-GNNWG-7TYQS-QSKT5-Y4U62-MWFZL' // 必填
			// });
			uni.chooseLocation({
				success: async function(res) {
					let locationObj = captureLocation(res);
					console.log(res);
					that.latitude = res.latitude;
					that.longitude = res.longitude;
					that.province = locationObj.province;
					that.city = locationObj.city;
					that.county = locationObj.county;
					console.log(that.province + that.city + that.county, "返回地址");
					uni.request({
						url: 'https://wis.qq.com/weather/common?source=pc&weather_type=observe%7Cforecast_1h%7Cforecast_24h%7Cindex%7Calarm%7Climit%7Ctips%7Crise',
						data: {
							//参数
							province: that.province,
							city: that.city,
							county: that.county,
						},
						header: {
							// 'Content-Type': 'application/x-www-form-urlencoded'
							'Content-Type': 'application/json' //自定义请求头信息
						},
						method: 'GET', //请求方式，必须为大写
						success: (res) => {
							console.log('接口返回------', res);
							that.res = res;
						}
					});
					// uni.request({
					// 	url: 'https://api.seniverse.com/v4?fields=weather_hourly_1h',
					// 	data: { //参数
					// 		key: 'SV6Z3XO57V6h4SyJb',
					// 		locations: that.latitude + ":" + that.longitude,
					// 	},
					// 	header: {
					// 		// 'Content-Type': 'application/x-www-form-urlencoded'
					// 		'Content-Type': 'application/json' //自定义请求头信息
					// 	},
					// 	method: 'GET', //请求方式，必须为大写
					// 	success: (res) => {
					// 		console.log('接口返回------', res);
					// 	}
					// })
				},
				fail: function(res) {
					console.log(res, "报错")
				}
			})
			const captureLocation = (res) => {
				// console.log('res', res);
				var regex = /^(北京市|天津市|重庆市|上海市|香港特别行政区|澳门特别行政区)/;
				var province = [];
				var addressBean = {
					province: null,
					county: null,
					city: null,
					address: null,
				};

				function regexAddressBean(address, addressBean) {
					// console.log('address', address);
					// console.log('addressBean', addressBean);
					regex = /^(.*?[市]|.*?[州]|.*?地区|.*?特别行政区)(.*?[区]|.*?[市]|.*?[县])(.*?)$/g;
					var addxress = regex.exec(address);
					//console.log('addxress', addxress);
					addressBean.city = addxress[1];
					addressBean.county = addxress[2];
					addressBean.address = addxress[3] + '(' + res.name + ')';
					// console.log(addxress);
				}
				if (!(province = regex.exec(res.address))) {
					regex = /^(.*?(省|自治区))(.*?)$/;
					province = regex.exec(res.address);
					addressBean.province = province[1];
					regexAddressBean(province[3], addressBean);
				} else {
					addressBean.province = province[1];
					regexAddressBean(res.address, addressBean);
				}
				return addressBean;
			};
			// uni.getLocation({
			// 	type: 'gcj02', //国家测绘局坐标
			// 	success: (res) => {
			// 		console.log("当前位置的经度：", res.longitude);
			// 		console.log("当前位置的纬度：", res.latitude);
			// 		that.latitude = res.latitude;
			// 		that.longitude = res.longitude;
			// 		that.markers[0].latitude = that.latitude;
			// 		that.markers[0].longitude = that.longitude;
			// 		qqmapsdk.reverseGeocoder({
			// 			location: {
			// 				latitude: that.latitude,
			// 				longitude: that.longitude
			// 			},
			// 			success: function(res) {
			// 				console.log('省市区:', res.result.address);
			// 			},
			// 			fail: function(res) {
			// 				console.log(res);
			// 			},
			// 			complete: function(res) {
			// 				console.log(res);
			// 			}
			// 		});
			// 	}
			// })
		},
		methods: {}
	}
</script>

<style>
	.map {
		width: 100%;
		height: 500rpx;
	}

	::-webkit-scrollbar {
		width: 0;
		height: 0;
		color: transparent;
	}
</style>
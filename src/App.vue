<template>
	<div class="appwrapper">

		<img src="image.jpg" class="headerimage"/>

		<div class="wrp">

			<div class="caption">TEST APP</div>
			<div class="content">
				<div class="action">
					<button class="bstyle" @click="getAccount">Get user Address</button>
				</div>

				<div class="action" v-if="address">
					<button class="bstyle" @click="getUserInfo">Get user Profile</button>
				</div>

				<div class="action">
					<button class="bstyle" @click="getzaddress">Get wallet address</button>
				</div>

				<div class="action">
					<button class="bstyle" @click="createChatRoom">Create chat room</button>
				</div>

				<div class="action">
					<button class="bstyle" @click="share">Share</button>
				</div>

				<div class="action">
					<button class="bstyle" @click="sendMessage" :disabled="lastChatRoom ? false : true">Send message in chat (last created room)</button>
				</div>

				<div class="action">
					<button class="bstyle" @click="getNodeInfo">Get node info</button>
				</div>

				<div class="action">
					<button class="bstyle" @click="makePayment">Make Payment</button>
				</div>

				<div class="action">
					<button class="bstyle" @click="requestPermissions">Request permissions [messaging]</button>
				</div>

				<div class="action">
					<button class="bstyle" @click="alertMessage">Alert Message</button>
				</div>

				<div class="action">
					<button class="bstyle" @click="openSettings">Open application settings page</button>
				</div>

				<div class="action">
					<button class="bstyle" @click="imageFromMobileCamera">Image From Mobile Camera</button>
				</div>

				<div class="action">
					<button class="bstyle" @click="gotoRandomRoute">Application Internal Link</button>
				</div>

				
			</div>

			<div class="subcaption">Additional (test)</div>

			<div class="content">
				<div class="action">
					<input type="file" />
				</div>
				<div class="action">
					<router-link to="test">Test vue link</router-link>
				</div>

				<div class="action">
					<a href="https://www.developer.com/">https://www.developer.com/</a>
				</div>
			</div>
		</div>
	</div>

	<div class="routeWrapper">
		<div class="caption">
			<span>Current route</span>
		</div>

		<div class="route">
			{{route}}
		</div>
	</div>

	<div class="emitted" v-if="emitted.length">

		

		<div class="caption">
			<span>Events</span>
		</div>

		<div class="events">
			<div class="event" v-for="(e, i) in emitted" :key="i">
				<div class="type">
					{{e.type}}
				</div>

				<div class="time">
					{{e.date}}
				</div>

				<div class="json">
					{{e.data}}
				</div>
			</div>
		</div>
		
	</div>

	<div class="lastresult" v-if="lastresult">
		<div class="result">{{lastresult}}</div>
		<div class="close">
			<button class="bstyle" @click="clearLastResult">Close</button>
		</div>
	</div>

	<div class="localstorage">
		{{localstorage}}
	</div>

</template>

<script>

function getRandomInt(min, max) {
    min = Math.ceil(min);
    max = Math.floor(max);
    return Math.floor(Math.random() * (max - min + 1)) + min;
}

function makeid(length) {
    let result = '';
    const characters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
    const charactersLength = characters.length;
    let counter = 0;
    while (counter < length) {
      result += characters.charAt(Math.floor(Math.random() * charactersLength));
      counter += 1;
    }
    return result;
}

export default {
	name: 'App',
	components: {
		
	},

	data : function(){
		return {
			sdk : null,
			lastresult : '',		
			address : '',
			emitted : [],
			localstorage : '',
			locale : '',
			lastChatRoom : '',
			applicationInfo : null
		}
	},

	computed : {
		route : function(){
			return this.$router.currentRoute.value.href
		}
	},

	mounted : async function(){

		/// required		

		this.sdk = new window.BastyonSdk()

		this.sdk.init().then((applicationInfo) => {
			this.applicationInfo = applicationInfo
		})

		this.sdk.emit('loaded')

		this.sdk.on('changestate', (data) => {
			console.log("getroute", this.sdk.getroute(data))
			this.$router.push(this.sdk.getroute(data))
		})

		////

		this.sdk.on('action', (d) => {

			this.emitted.push({
				type : 'action',
				data : JSON.stringify(d),
				date : new Date(),
			})
		})

		this.sdk.on('balance', (d) => {

			this.emitted.push({
				type : 'balance',
				data : JSON.stringify(d),
				date : new Date(),
			})
		})

		this.sdk.on('state', (d) => {

			this.emitted.push({
				type : 'state',
				value : d
			})
		})

		this.sdk.on('keyboard', ({height}) => {

			this.emitted.push({
				type : 'keyboard',
				height : height
			})
		})

		this.sdk.get.appinfo().then(({locale}) => {

			console.log('locale', locale)

			this.locale = locale
		})

		if (this.sdk.proxyRequests){
			await this.sdk.proxyRequests()
		}

		setInterval(async() => {

			let url = 'https://matrix.2.pocketnet.app/_matrix/client/versions'

			let response = await fetch(url);

			if (response.ok) { 
				let json = await response.json();

				console.log(json)

			} else {
				alert("error HTTP: " + response.status);
			}
		}, 10000)
		
		/*this.sdk.on('test', (d) => {
			console.log("test data", d)
		})*/

		/*sdk.get.account().then(address => {
			console.log("applications: useraddress", address)
		})*/

		/*sdk.rpc('getnodeinfo').then(data => {
				console.log("applications: NODEINFO", data)
		})*/

		this.checkLocalStorageAccess()
	},

	methods : {
		share : function(){
			this.sdk.helpers.share({
				path : '/url?article=4',
				/*sharing : {
					images : ['https://peertube341.pocketnet.app/images/8fc002607bc3e2b4f29d6a3dae764798/8fc002607bc3e2b4f29d6a3dae764798-original.jpg'],
					title : 'Article',
					html : {
						body : '<p>Article text</p>',
						preview : '<p>Article text preview</p>'
					},

					text : {
						body : 'Article text',
						preview : 'Article text preview'
					}
				}*/
			})
		},
		gotoRandomRoute : function(){
			this.$router.push(makeid(10) + '?' + makeid(2) + '=' + getRandomInt(100, 200))
			
		},
		setLastResult : function(){

		},
		clearLastResult : function(){
			this.lastresult = ''
		},

		getzaddress: function(){
			this.sdk.get.zaddress().then((address) => {
				console.log('address555', address)
				this.lastresult = 'getzaddress: ' + address

			}).catch(e => {
				this.lastresult = e
			})
		},

		openSettings: function(){

			this.sdk.helpers.opensettings().then(() => {
				this.lastresult = 'opensettings: success'

			}).catch(e => {
				this.lastresult = e
			})
		},

		openChatRoom : function(roomid){
			this.sdk.chat.openRoom(roomid).then(() => {

				this.lastresult = 'openChatRoom: / ' + roomid

			}).catch(e => {
				this.lastresult = e
			})
		},


		createChatRoom : function(){
			var users = ['PH6Sn3Kb8y2oLdgoysyGL5nhdr6jZMNjhv', 'PKEMJ5pJ3tUMdSSnJNBNG5iWTHJm33duCk', 'PBZ3tmNb31JwzPUS6TuE2St8ZWHi3ks2tt', 'PPcRoWh5oAZgAdqL2spMmRyE1BB9gSWgPH']
			
			if (this.sdk.test){
				users = ['THhQUbviS194HoCzPnLZ5ADgNs15QQReyf', 'T9yF79sfm59dkVa6a2ZJFa3hQNr48R1tmq', 'T9yEbCJe7PmPmiXh4hLFPfTtV5HQNjSFpu', 'T9yEPapEnfhQZxvu3nb5pZAPEivKseqkFg', 'TDGUg8Uohj4Hw9DbUJkZynswo77E42WN8T']
			}

			var count = getRandomInt(1, 2)

			var chatusers = [];

			for(var i = 0; i < count; i++){
				if(users.length){
					var index = getRandomInt(0, users.length - 1)
					chatusers.push(users[index])
					users.splice(index, 1);
				}
			}

			this.sdk.chat.getOrCreateRoom({users : chatusers, parameters : {equal : true}}).then(({roomid}) => {

				console.log('roomid', roomid)

				this.lastresult = 'getOrCreateRoom: success / ' + roomid

				this.lastChatRoom = roomid

				this.openChatRoom(roomid)

			}).catch(e => {
				this.lastresult = e
			})

		},

		sendMessage : function(){

			console.log('this.lastChatRoom', this.lastChatRoom)
			console.log('this.applicationInfo', this.applicationInfo)

			if(!this.lastChatRoom) return
			if(!this.applicationInfo) return

			var roomid = this.lastChatRoom


			var content = {};

				//content.images = [imagesForShareBase64];
				//content.messages = [message1,message2];


			content.messages = ['Hello!', 'This is link from my mini application: ' + this.sdk.get.applink()]
			
			console.log('content', content)

			this.sdk.chat.send({
				
				roomid,
				content

			}).then(() => {

				this.lastresult = 'sendMessage: success / ' + roomid

				this.openChatRoom(roomid)

			}).catch(e => {
				this.lastresult = e
			})

		},

		imageFromMobileCamera : function(){

			this.sdk.get.imageFromMobileCamera().then((images) => {
				this.lastresult = 'imageFromMobileCamera: success (console.log)'
				console.log(images)

			}).catch(e => {
				this.lastresult = e
			})
		},

		alertMessage: function(){

			this.sdk.helpers.alert('Test message').then(() => {
				this.lastresult = 'alert: success'

			}).catch(e => {
				this.lastresult = e
			})
		},

		requestPermissions : function(){

			this.sdk.request.permissions(['messaging']).then(() => {
				this.lastresult = 'messaging: granted'
			}).catch(e => {
				this.lastresult = e
			})
		},

		getAccount : function(){

			this.sdk.get.account().then(({address, signature}) => {
				console.log(signature)
				this.lastresult = 'user address: ' + address

				this.address = address
			}).catch(e => {
				this.lastresult = e
			})
		},

		getNodeInfo : function(){
			this.sdk.rpc('getnodeinfo').then(data => {
				this.lastresult = JSON.stringify(data, null, '\t')
			}).catch(e => {
				this.lastresult = e
			})
		},

		getUserInfo : function(){

			if(!this.address) return

			this.sdk.rpc('getuserprofile', [[this.address]]).then(data => {
				this.lastresult = JSON.stringify(data, null, '\t')
			}).catch(e => {
				this.lastresult = e
			})
		},

		makePayment: function(){

			var data = {
				'recievers' : [{
					address : 'PR7srzZt4EfcNb3s27grgmiG8aB9vYNV82',
					amount : 0.01
				}], 
				'feemode' : 'include', 
				'message' : ''
			}

			this.sdk.payment(data).then(data => {
				this.lastresult = JSON.stringify(data, null, '\t')
			}).catch(e => {
				this.lastresult = e
			})
		},

		checkLocalStorageAccess : function(){

			
			function allStorage() {

				var values = [],
					keys = Object.keys(localStorage),
					i = keys.length;

				while ( i-- ) {
					values.push( localStorage.getItem(keys[i]) );
				}

				return values;
			}

			localStorage['checkLocalStorageAccess'] = true
			this.localstorage = allStorage()
		}
	}
}
</script>

<style>
.routeWrapper{
	padding : 0.5em;
	margin-bottom: 3em;
}
.localstorage{
	display: none;
}
.event{
	padding : 0.5em;
	border-bottom : 1px solid #2c3e50;
	overflow: hidden;
}
.event .type{
	font-weight: 700;
}

.event .date{
	opacity: 0.87;
}

.event .json{
	margin-top: 1em;
}

.caption span{
	font-weight: 700;
}

.emitted{
	margin-top: 2em;
}
#app {
	
}
.wrp{
	padding: 0.5em;
	padding-bottom: 500px;
}
.action{
	display: flex;
	align-items: center;
	margin-bottom: 1em;
}
.headerimage{
	width: 100%;
}
.lastresult{
	position: fixed;
	bottom : 0;
	left : 0;
	right: 0;
	background: #f5f5f5;
	padding : 1em;
	max-height: 50%;
	border-top-left-radius: 12px;
	border-top-right-radius: 12px;
	overflow-y: overlay;
	overflow-x: hidden;
	

}
.lastresult .result{
	padding-bottom: 60px;
}
.lastresult .close{
	padding : 1em;
	position : fixed;
	left: 0;
	bottom : 0;
	right: 0;
	z-index: 2;
	background: #f5f5f5;
}

.caption{
	font-size: 1.5em;
	margin-bottom: 1em;
	margin-top: 1em;
	font-weight: 700;
}

.subcaption{
	margin-bottom: 1em;
	font-weight: 700;
}
</style>

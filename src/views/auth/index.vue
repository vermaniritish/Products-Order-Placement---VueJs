<template>
	<div class="bottom-modal-wrap">
    	<div class="bottom-modal" v-if="!otpModal">
			<button class="close" v-if="!nocross" v-on:click="hide()"><i class="far fa-times"></i></button>
			<div class="close-area row">
				<div class="col-sm-12"></div>
			</div>
			<div class="row">
				<div class="col-sm-12">
					<h3>Welcome!</h3>
					<p class="text-muted">Please enter your phone number to login.</p>
				</div>
			</div>
			<div class="row">
				<div class="col-sm-12 mb-2">
					<div class="pn-select my-2" id="js_pn-select" style="--prefix-length: 2">
						<button class="pn-selected-prefix" aria-label="Select phonenumber prefix" id="js_trigger-dropdown" tabindex="1">
							<img class="pn-selected-prefix__flag" id="js_selected-flag" src="https://flagpedia.net/data/flags/icon/36x27/in.png">
						</button>
						<div class="pn-input">
							<div class="pn-input__container">
							<input class="pn-input__prefix" value="+91" type="text" name="phonenumber-prefix" id="js_number-prefix" tabindex="-1">
							<input class="pn-input__phonenumber" id="js_input-phonenumber" type="tel" name="phonenumber" pattern="\d*" value="" placeholder=" " autocomplete="nope" required="" maxlength="10" tabindex="0" v-model="phonenumber">
							</div>
						</div>
					</div>
					<p class="text-muted">Securing your personal information is our priority.</p>
				</div>
				<div class="col-sm-12">
					<button class="btn btn-primary" v-on:click="submit"><i v-if="submitting" class="far fa-circle-notch fa-spin"></i> Continue</button>
				</div>
			</div>
		</div>
		<div class="bottom-modal" v-else>
			<button class="close" v-on:click="hide()"><i class="far fa-times"></i></button>
			<div class="close-area row">
				<div class="col-sm-12"></div>
			</div>
			<div class="row">
				<div class="col-sm-12">
					<h3>Verify Phone Number</h3>
					<p class="text-muted">Code has sent to +91-{{this.phonenumber}}</p>
				</div>
			</div>
			<div class="row">
				<div class="col-sm-12 my-2">
					<div class="otp-input-fields">
						<input type="number" class="otp__digit otp__field__1" v-model="otp1" maxlength="1">
						<input type="number" class="otp__digit otp__field__2" v-model="otp2" maxlength="1">
						<input type="number" class="otp__digit otp__field__3" v-model="otp3" maxlength="1">
						<input type="number" class="otp__digit otp__field__4" v-model="otp4" maxlength="1">
					</div>
					<p class="text-danger" v-if="tmpOtp">The temp auth OTP is {{tmpOtp}}</p>
					<p class="text-end"><a href="javascript:;" v-on:click="resendCode">Resend Code</a></p>
				</div>
				<div class="col-sm-12">
					<p class="text-danger text-center" v-if="error">{{ error }}</p>
					<button class="btn btn-primary" v-on:click="verify"><i v-if="submitting" class="far fa-circle-notch fa-spin"></i> Verify</button>
				</div>
			</div>
		</div>
	</div>
</template>
  
<script>
import config from '../../config';

export default {
	name: 'LoginModal',
	props: {
    	show: Boolean,
		hide: Function,
		user: Function,
		nocross: Boolean
	},
	components: {
	},
	data() {
		return {
			information: null,
			phonenumber: ``,
			submitting: false,
			otpModal: false,
			error: null,
			tmpOtp: null,
			otp1: ``,
			otp2: ``,
			otp3: ``,
			otp4: ``,
		};
	},
	methods: {
		async submit() {
			if(!this.phonenumber || this.submitting) return false;
			this.submitting = true;
			let response = await fetch(config.api_url + '/auth/login', {
				method: 'POST',
				headers: {
					'Content-Type': 'application/json'
				},
				body: JSON.stringify({
					phonenumber: this.phonenumber,
					name: this.information && this.information.name ? this.information.name : null
				})
			});
			response = await response.json();
			this.submitting = false;
			if(response && response.status && response.hash)
			{
				this.token = response.hash;
				this.tmpOtp = response.otp;
				this.initOtpModal();
			}
		},
		async verify() {
			console.log(!this.token , this.submitting , this.otp1.toString().trim() === '' , this.otp2.toString().trim() === '' , this.otp3.toString().trim() === '' , this.otp4.toString().trim() === '', 'yes')
			if(!this.token || this.submitting || this.otp1.toString().trim() === '' || this.otp2.toString().trim() === '' || this.otp3.toString().trim() === '' || this.otp4.toString().trim() === '') return false;
			this.error = null;
			this.submitting = true;
			let response = await fetch(config.api_url + '/auth/second-auth/' + this.token , {
				method: 'POST',
				headers: {
					'Content-Type': 'application/json'
				},
				body: JSON.stringify({
					otp: `${this.otp1}${this.otp2}${this.otp3}${this.otp4}`
				})
			});
			response = await response.json();
			this.submitting = false;
			if(response && response.status) {
				localStorage.setItem('auth', JSON.stringify(response.user));
				localStorage.removeItem('information');
				this.user(response.user);
				this.$emit('hide');
				this.hide();
			}
			else if(response.message) {
				this.error = response.message ? response.message : 'Something went wrong.'
			}
		},
		initOtpModal() {
			this.otpModal = true;
			setTimeout(() => {
				var otp_inputs = document.querySelectorAll(".otp__digit");
				var mykey = "0123456789".split("");
				var that = this;
				otp_inputs.forEach((_) => {
					_.addEventListener("keyup", function(event){
						let current = event.target
						let index = parseInt(current.classList[1].split("__")[2])
						current.value = event.key
						
						if(event.keyCode == 8 && index > 1){
							current.previousElementSibling.focus()
						}
						if(index < 4 && mykey.indexOf(""+event.key+"") != -1){
							var next = current.nextElementSibling;
							next.focus()
						}

						if(index >= 4) {
							current.blur();
							that.verify();
						}
					})
				});
				otp_inputs[0].focus();
			}, 300);
		},
		resendCode() {
			this.otpModal = false;
			this.error = null;
		}
	},
	mounted() {
		let information = localStorage.getItem('information');
		this.information = information ? JSON.parse(information) : null;
		document.body.style.overflow = 'hidden';
		document.getElementById('js_input-phonenumber').focus();
	},
};
</script>
  
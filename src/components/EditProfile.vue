<template>
  <div class="contentPanel">
    <div v-if="haserror" class="errorMsg">{{errormsg}}</div>
    <div v-if="regsuccess" class="successMsg"><strong>Congratulations!</strong> Your profile successfully updated.</div>
  
    <div class="titlePanel">
      <div class="titleBar">Edit Profile</div>
    </div>
    <div class="loginWrap profileWrap">
  
      <div class="loginPanel">
        <div class="userImage" v-bind:style="{ backgroundImage: 'url(' + image + ')' }"><a v-on:click="uploadfile" href="javascript:void(0);"><i class="right fa fa-pencil"></i></a></div>
        <input type="file" ref="userimg" @change="fileselected" id="user-img" class="user-img" style="display:none;">
        <form action="#">
          <div class="input-field">
            <input placeholder="Email" v-on:keyup.enter="updteprofile" v-model="email" disabled id="email" type="text" class="validate">
          </div>
          <div class="input-field">
            <input placeholder="First name" v-on:keyup.enter="updteprofile" v-model="fname" id="fName" type="text" class="validate">
          </div>
          <div class="input-field">
            <input placeholder="Last name" v-on:keyup.enter="updteprofile" v-model="lname" id="lName" type="text" class="validate">
          </div>
          <div class="input-field">
            <input placeholder="Phone" v-on:keyup.enter="updteprofile" v-model="phone" id="phone" type="text" class="validate">
          </div>
          <div class="input-field">
            <input placeholder="City" v-on:keyup.enter="updteprofile" v-model="city" id="city" type="text" class="validate">
          </div>
          <div class="input-field">
            <input placeholder="Country" v-on:keyup.enter="updteprofile" v-model="country" id="country" type="text" class="validate">
          </div>
          <div class="input-field">
            <input placeholder="Lattitude" v-on:keyup.enter="updteprofile" v-model="lat" id="lat" type="text" class="validate">
          </div>
          <div class="input-field">
            <input placeholder="Language" v-on:keyup.enter="updteprofile" v-model="lng" id="lng" type="text" class="validate">
          </div>
          <div class="input-field">
            <a v-on:click="updteprofile" href="javascript:void(0);" class="btn btnPrimary waves-effect waves-light">{{action}}</a>
          </div>
        </form>
      </div>
  
  
    </div>
  </div>
</template>

<script>
  export default {
  
    data() {
      return {
        haserror: false,
        errormsg: '',
        action: 'Update Profile',
        regsuccess: false,
        auth: '',
        userid: '',
        image: '',
        email: '',
        fname: '',
        lname: '',
        phone: '',
        city: '',
        country: '',
        photourl: '',
        lat: '',
        lng: '',
      }
    },
    mounted() {
      this.initpage()
    },
    methods: {
  
      // Validations
      validateNumber() {
  
        if (!this.phone || isNaN(this.phone)) {
          this.errormsg = "Inavlid Phone Number!"
          this.haserror = true
          return false;
        }
  
        return true
      },
      ValidateEmail: function() {
        if (/^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{2,3})+$/.test(this.email)) {
          return (true)
        }
        return (false)
      },
  
      //Helpers
      uploadfile: function() {
        var img = this.$refs.userimg
        img.click()
      },
      fileselected: function(e) {
        console.log(e)
        let files = e.target.files || e.dataTransfer.files;
        if (!files.length) {
          return;
        }
        this.createImage(files[0]);
        console.log(this.image)
      },
      createImage: function(file) {
        let image = new Image();
        let reader = new FileReader();
  
        reader.onload = (e) => {
          this.image = e.target.result;
        };
        reader.readAsDataURL(file);
      },
      // previewFile: function() {
      //   var preview = document.getElementById('user-img-input'); //selects the query named img
      //   var file = document.querySelector('input[type=file]').files[0]; //sames as here
      //   var reader = new FileReader();
  
      //   reader.onloadend = function() {
      //     preview.src = reader.result;
      //   }
  
      //   if (file) {
      //     reader.readAsDataURL(file); //reads the data as a URL
      //   } else {
      //     preview.src = "";
      //   }
      // },
      initpage: function() {
  
        if (this.$session.has("userid") == false) {
          this.$router.push("/login")
          return
        }
  
        let token = 'Bearer ' + this.$session.get("token")
        const auth = {
          headers: {
            'Authorization': token
          }
        }
  
        let url = "user/" + this.$session.get("userid") + "/detail"
        var model = this
        this.$http.get(url, auth).then(function(data) {
  
          console.log(data.body)
          data = data.body
          if (data['status'] === true) {
  
            data = data.data
            model.email = data.email
            if(data.photoURL !== null) model.image = data.photoURL
            else model.image = "../assets/images/favicon.png"
            model.fname = data.first_name
            model.lname = data.last_name
            model.phone = data.phone
            model.city = data.city
            model.country = data.country
            model.lat = data.lat
            model.lng = data.lng
  
          } else {
  
            model.haserror = true
            model.errormsg = data.message
          }
  
        })
      },
      updteprofile: function() {
  
        this.haserror = false
  
        if (!this.validateNumber()) {
          return;
        }
  
        var model = this
        model.action = "Please wait"
        let token = 'Bearer ' + this.$session.get("token")
        let userid = this.$session.get("userid")
        const auth = {
  
          headers: {
            'Authorization': token
          }
        }
  
        let imageform = new FormData(this.image)

        
        let url = "user/update"

        this.$http.put(url, {
  
          user_id: userid,
          email: model.email,
          photoURL: imageform,
          first_name: model.fname,
          last_name: model.lname,
          phone: model.phone,
          city: model.city,
          country: model.country,
          lat: model.lat,
          lng: model.lng
  
  
        }, auth).then(function(data) {
          data = data.body
          console.log(data)
          if (data['status'] === true) {
            model.action = "Profile Updated"
            model.regsuccess = true
            model.errormsg = false
            this.$router.push('/profile')
          } else {
            model.haserror = true
            model.errormsg = data.message
  
          }
        })
      }
    }
  }
</script>



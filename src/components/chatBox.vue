<!-- zone affichage message + input-->
<!-- ===================== template ===================== -->
<template>
	<div id="chatbox">
		<div id="tchat">
			<button @click="ancienMessageAfficher">ancien message</button>
			<button v-if="listeMessagesLongueur > 30" @click="ancienMessageSupprimer">moins</button>
			<ul class="listeMessage">
				<li id="messageCSS" v-for="message in listeMessages">
					<div v-if="message[1] == user" class="messageMoi"><b class="userCSS">moi</b><span><div v-html="message[0]" v-linkified /></span></div>
					<div v-else class="messageAutre"><b class="userCSS">{{ message[1] }}</b><span ><div v-html="message[0]" v-linkified /></span></div></li>
			</ul>
		</div>
		<div class="envoyerCSS">
			<img src="../assets/icons/paperclip.png" alt="">
			<img src="../assets/icons/emote.png" alt="">
			<input type="text" v-model="messageEnvoyer" @keyup.enter="envois">
			<img src="../assets/icons/paperplane.png" alt="">
		</div>
	</div>
</template>

<!-- ====================== script ========================== -->

<script>

import { db } from '../main'
import firebase from 'firebase'





export default { 

name: 'chatBox',
props: ['sectionActive'],

data () {
	return	{
		listeMessages: [],
		listeMessagesLongueur: 0,
		limitMessage : 20,
		user: '',
		messageEnvoyer: '',
		listeMembres: []
		
		
	}
},

methods: {
	afficherMessage () {
	db.collection('sections/' + this.sectionActive + '/messages').orderBy("envois", "desc").limit(this.limitMessage).onSnapshot(querySnapshot => {

			this.listeMessages = []
        querySnapshot.forEach(doc => {
        	var infoMsg = [doc.data().text,doc.data().expediteur]
           this.listeMessages.push(infoMsg)
        });

        this.listeMessages.reverse()
        this.listeMessagesLongueur = this.listeMessages.length
        // window.scroll(0,10000)
        
        //======== vu =============

        //recup vu
        db.collection('sections').doc(this.sectionActive).get().then(doc => {
   			if (doc.exists) {	
   					this.listeMembres = {}
   					this.listeMembres = doc.data().vu
   					//modif vu
				        this.listeMembres[firebase.auth().currentUser.uid] = true
				        db.collection('sections').doc(this.sectionActive).update({
							vu: this.listeMembres
						})	
    	}
		})

        


        
    });
    
},
ancienMessageAfficher(){
	this.limitMessage += 10
	this.afficherMessage()
	
},
ancienMessageSupprimer(){
	this.limitMessage += -10
	this.afficherMessage()
	
	
},

envois () {
		db.collection('sections/' + this.sectionActive + '/messages').add({
    envois: new Date,
    text: this.messageEnvoyer,
    expediteur: firebase.auth().currentUser.displayName
		})
		.then(function(docRef) {
		    console.log("Document written with ID: ", docRef.id);
		})
		.catch(function(error) {
		    console.error("Error adding document: ", error);
		});
	this.messageEnvoyer = ''
	//notification
		this.recuperationMembres()
		db.collection('sections').doc(this.sectionActive).update({
			vu: this.listeMembres
		})
		




},

	recuperationMembres() {
		this.listeMembres = {}
		var docRef = db.collection("sections").doc(this.sectionActive);

			docRef.get().then(doc => {

			    if (doc.exists) {
			        
			    	
			        if(doc.data().membres != undefined){
			        	this.listeMembres = {}
			        	this.listeMembres = doc.data().membres
			        }

			        else {
			        	db.collection("users").get()
						    .then(querySnapshot => {
						    	var membresGlobal ={}
						        querySnapshot.forEach(doc => {
						            membresGlobal[doc.id] = true
						        });
						        db.collection("sections").doc(this.sectionActive).update({membres: membresGlobal})
						        this.listeMembres = membresGlobal
						        console.log("ajout membres",this.listeMembres);

						    })
						    .catch(function(error) {
						        console.log("Error getting documents: ", error);
						    });

			        	
			        }


			       

			        // for(var membre in this.listeMembres){
			        // 	this.listeMembres[membre] = false;

			        // }



			       





			    } else {
			        // doc.data() will be undefined in this case
			        console.log("No such document!", this.sectionActive);

			    }
			}).catch(error => {
			    console.log("Error getting document:", error);
			});
	}

},

created (){
	this.afficherMessage()
	this.user = firebase.auth().currentUser.displayName
	
},

watch: {
  sectionActive: function() {
          this.afficherMessage ()


        }
  	
  
    
}



}




</script>

<!-- ====================== CSS ========================== -->

<style>
	#chatBox{
		width: 100%
	}


	#tchat {
		height: 80vh;
		width: auto;
		overflow-y: scroll;
		margin:auto;
		background-color: #1F303A;
		background-image: url('../assets/icons/mascotOpacity.png');
		background-repeat: no-repeat;
		background-position: center;

	}


	#messageCSS {
		padding:5px;
		width: 100%;
		list-style: none;
		color:white;
	}

	.userCSS {
	}
	.userCSS{
		color:#1C9EA0;
	}

	.envoyerCSS {
	margin: 0 auto;
	width: 100%;
	height: 10vh;
	background-color: #111B24;
	line-height: 10vh;




}

.envoyerCSS>img{
	margin-left:3px; 
}
.envoyerCSS>input{
	height: 55px;
	width: 72%;
	background-color: #2A3C48;
	border:none;
	border-radius: 12px;
	margin-left:3px;
	color:white; 
}


.messageMoi{
	text-align: right;
	margin: 5px;
	margin-right: 50px;

}
.messageAutre{
	margin:5px;
	margin-left: 15px;
}






/*lien*/

 /* unvisited link */
a:link {
    color: #63C3D1;

}

/* visited link */
a:visited {
    color: #63C3D1;
}

/* mouse over link */
a:hover {
    color: #63C3D1;
}

/* selected link */
a:active {
    color: #63C3D1;
} 

</style>
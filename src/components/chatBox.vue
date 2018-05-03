<!-- zone affichage message + input-->
<!-- ===================== template ===================== -->
<template>
	<div>
		<div id="tchat">
			<button @click="ancienMessageAfficher">ancien message</button>
			<button v-if="listeMessagesLongueur > 30" @click="ancienMessageSupprimer">moins</button>
			<ul class="listeMessage">
				<li class="messageCSS" v-for="message in listeMessages">
					<div v-if="message[1] == user" class="userCSS"><b>moi</b>: <span> {{ message[0] }}</span></div>
					<div v-else ><b>{{ message[1] }}</b>:<span > {{ message[0] }}</span></div></li>
			</ul>
		</div>
		<input class="envoyerCSS" type="text" v-model="messageEnvoyer" @keyup.enter="envois">
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
	#tchat {
		height: 600px;
		width: 500px;
		overflow: scroll;
		margin:auto;
	}
	.messageCSS {
		padding:5px;
		list-style: none;
	}

	.userCSS {
		margin-left: 200px;
	}

	.envoyerCSS {
	margin: 0 auto;
	width: 500px;

}

</style>
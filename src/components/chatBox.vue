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
		messageEnvoyer: ''
		
		
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
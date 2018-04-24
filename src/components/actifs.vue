<!-- zone affichage message -->
<!-- ===================== template ===================== -->
<template>
	<div class="actifsCSS">
		<div class="actifs" v-for="actif in actifs">{{actif}}</div>
	</div>
</template>

<!-- ====================== script ========================== -->

<script>

import { db } from '../main'
import firebase from 'firebase'


export default { 

name: 'actifs',

data () {
	return	{
		
		actifs: ['actifs']
		
	}
},

methods: {
	afficherActifs () {
		console.log('ok actifs');
		db.collection("users").where('actif','==',true).onSnapshot(querySnapshot => {
		this.actifs = ['actifs']	
        querySnapshot.forEach(doc => {
           this.actifs.push(doc.data().name)
        })
		})
		}
},

created (){
	this.afficherActifs()
	
	
}




}




</script>

<!-- ====================== CSS ========================== -->

<style>
	.actifsCSS {
		position: absolute;
	}
</style>
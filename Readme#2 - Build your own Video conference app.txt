Tech: Vanilla JS +  Firebase (Signalling server)

Objective:	
User #1 access webcam
Create call --> to FireStore
Create peer connection instance in browser in Firestore 	//Web RTC negotiation process
generate unique id -> other user can access this
write answer details in same document

Peer#1 and Peer2 write ICE Candidates to DB

subcolection #1  : offer candidates
subscollection#2 : answer candidates
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

1) JS Code: Use Vite Build tool by Vue.js
	npm init @vitejs/app

//What output? gives web project , where can install dependencies wo worrying about webpack etc	 

2) Install firebase
	contains firestore DB	//Backend signalling server	 
	//easy to listen to updates of DB realtime
	//Traditional BD --> might have to implement websockets

3) Initialize firestore in test mode
	create web project from settings
	copy project credentials

	' fit.
const firebaseConfig = { It
apiKey: "AIzaSyD3pthvIdXNyK92Mt20ij2_3x2467ij",
authDomain: "fireship—demos.firebaseapp.com",
databaseURL: "https://fireship—demos.firebaseio.com",
projectId: "fireship—demos",
storageBucket: "fireship-demos.appspot.com",
messagingSenderId: "454289764797",
appId: "1:454289704797zweb:495ad449cdabffa0940ab4"
} :

4) What JS project structure looks like

WEBRTC-DEMO
> @ node_modu|es
Q} .gitignore
é‘vé favicon.svg
l5 index.htm|
JS main.js
m package-lockjson
m packagejson
3 style.css



//main.js
import './style.css';

import firebase from 'firebase/app';

import 'firebase/firestore';				//import firebase

konst firebaseConfig = {				//firebase config
apiKey: 'AIzaSyD3pthvIdXNyK92Mt2Qij2_3x2407ij',
authDomain: 'fireship—demos.firebaseapp.com',
databaseURL: 'https://fireship—demos.firebaseio.com',
projectId: 'fireship-demos',
storageBucket: 'fireship-demos.gppépgt.com',
messagingSenderId: '454289704797',
appId: '1:454289704797:web:405ad440cdabffac940ab4',

};

if (lfirebase.apps.length) { 				//Initialize with firebase config
firebase.initializeApp(firebaseConfig);
}


let pc = new RTCPeerConnection();			//Initialize 3 pieces of global state		//What are these? value we will share between components (React/Angular)
//emits events we can listen to update DB + add media stream to connection
//generates ICE candidates , (have to tell which STUN server to use)
//we use free google stun servers

const servers = {
iceServers: [
{
	urls: ['stun:stunl.l.google.com:19302', ‘stunzstunZ.l.google.com:19302‘],
},
],
	iceCandidatePoolSize: 10,
};
// Gtobat State
let pc = new RTCPeerConnection(servers);

//values for localStream and remoteStream ---> What are these? video streams from our and user's webcam
let localStream = null;
let remoteStream = null;


//Use imperative DOM API (Since working with Vanilla JS) 
//grab some elements from html	
//html has video elements for: a) show video feeds b) interaction buttons in UI etc

const webcamButton = document.getElementById('webcamButton');
const webcamVideo = document.getElementById('webcamVideo');
const callButton = document.getElementById('callButton');
const callInput = document.getElementById('callInput');
const answerButton = document.getElementById('answerButton');
const remoteVideo = document.getElementByIdI'remoteVideo'l;
const hangupButton = document.getElementById('hangupButton');


//Event handler for click event on button	//How to obtain stream from user's webcam? await navigator media devices -> getUserMedia()
		set video and audio to true
		This is how dialog box for //askuser for permission to access webcam //appears
		//finally promise will resolve to media stream object
webcamButton.onclick = async () => {
	localStream = await navigatorlmediaDevices.getUserMedia({videp: true, audio: true});
	remoteStream = new MediaStream();		//setup remote stream	//Empty media stream right now
	
	//take 2 streams -> make available on peer connection -> show them on video elements in DOM
	localStream.getTracks().forEach((track)) => {
		pc.addTrack();				//for each track call peer connection add track  //Pushing Audio/Video to Peer connection
	});

	pc.ontrack = event => {
		//remote stream is empty, peer connection wil update it //we just listen to ontreack event on peer connection
		//get tracks from stream --> loop and add them --> add them to remote stream when they come in
		event.streams[0].getTracks()forEach(track => {remoteStream.addTrack(track);		//add them to rmote stream when they come in	 
	});
	webcamVideo.srcObject = localStream;	//apply streams to video elements on DOM , how? srcObject //set it equal to stream
	remoteVideo.srcObject = remoteStream;	//

	//What done? we have a way to manage local and remote stream through peer connection


//How to make peer connection? Signalling in Firestore


//a) Create an offer : async() in callButton
//Save to DB

callButton.onclick = async() => {
	const callDoc =  firestore.collection('calls').doc();  		//this document manages answer and offer from both users
								//offer candidates and answer candidates //sub collection under the document
														//these contain all candidates for each of the users

	const offerCandidates = callDoc.collection('offerCandidates');
	const answerCandidates = callDoc.collection('answerCandidates');
	
	callInput.value = callDoc.id;			//generating ice candidates
	// Get candidates far catter, save to db		//What ice candidate contains? potentital IP address and  port pair  //used to establish actual peer to peer connection
	pc.onicecandidate = event => {
	 event.candidate && offerCandidates.add(event/candidate.toJSON());		//establish listener before making peer to peer connection
							//when event fire -> make sure candidate exists -> write data as JSON to  to offer candidate collection
	};






									//when referencing document iwthout id --> firebase auto generates an ID for us
									//This is how we answer call?
									//use ID to populate input in UI --> Use in browser tab or another user in world to answer call
	const offerDescription  = await pc.createOffer();			//returns offer description
	await pc.setLocalDescription(offerDescription);				//set it as local Description on  peer connection
		//what this contains? SDP value //session description value	 //What is this value? we save it in DB
		//Note here only it will start generating ICE candidates
			//What ICE Candidate contains? IP + port pair //Used to establish actual peer to peer connection
				//Have to listen to ice candidates  


	//Convert it into plain JS obeject
						//SDP object contains info about video codec etc (things needed to negotiate connection)
	const offer = {
	   sdp: offerDescription.sdp,
	   type: offerDescription.type,
	}; 
	
	//write to DB	
	await callDoc.set({offer});

//What achieved? We as caller making offer have saved our data to DB
//What left? how to listen to answer from user at other end? Listen for changes toc all document in firestore. onSnapshot()
	//What this method does? fire callback anytime document or DB changes
		//checks if peer connection doesnt have currentRemoteDescription or data does not have answer
			//set answer description on  peer connection locally

	//i.e listen to firend for answer -> when asnwer received -> update on peer connection


callDoc.onSnapshot((snapshot) => {
   const data = snapshot.data();
  if(!pc.currentRemoteDescription)  {
	const answerDescription = new RTCSessionDescription(data.answer);
	pc.setRemoteDescription(answerDescription);



 }

});

//What achieved? inital connection negotiated
  What left? listen for ice candidates from answering user
How? listen for updates to  answerCandidates collection  //firestore method .docChanges()	on query //can listen to only documents added to collection  

Every time new doc added -> create new ice candidate with doc data --> add candidate to peer connection

answerCandidates.onSnapshot(snapshot => {
    snapshot.docChanges().forEach(change) => {
        if(change.type === 'added')  {
	 const candidate = new RTCIceCandidate(change.doc.data());
	pc.addIceCandidate(candidate);
     }
  });
});

//What achieved? listen to updates from answer side		//answer call with unique id
//How to give answering user a way to actually answer call?
How to answer call? similar to initiating a call. 
Listen to a document on firestore  --->id should be same as one created by caller --> make reference to the doc + answerCandidates collection into ICE Candidate event

 

answterButton.onclick = async () => {
const callId = callInput.value;
const callDoc = firestore.collection('calls').doc(callId);
const answerCandidates = callDoc.collection('answerCandidates');

	//Listen to ICE Candidate event on peer connection -> update answerCandidate collection whenever new candidate generated
pc.onicecandidate = event => {
	event.candidate && answerCandidates.add(event.candidate.toJSON());
};

	//Fetch the call document from DB and grab the data
	//What it contains? offer data ---> we use it to set remote Description on peer connection
	

const callData = (await callDoc.get()).data();
const offerDescription = callData.offer;
await pc.setRemoteDescription(new RTCSessionDescription(offerDescription));
//generate answer locally ---> set local description as answer
const answerDescription = await pc.createAnswer();

//set localDescription as answer
await pc.setLocalDescription(answerDescription);

//set it up as plain object
const answer = {
	type: answerDescription.type,
	sdp: answerDescription.sdp,
};

//update on call document --> so that other use can listen to the answer
await callDoc.update({answer});


//Setup a listener on offer candidates collection	
//whenever new ice candidate added to collection --> create a ice candidate locally

offerCandidates.onSnapshot((snapshot) => {
snapshot.docChanges().forEach((change) => {
console.log(change)
if (change.type === 'added') {
 let data = change.doc.data();
pc.addIceCandidate(new RTCIceCandidate(data));
}
});
});

};
	



Objective achieved: Created video chat feature with web RTC.
What major work?
singalling data bw 2 users
//handles all complicated peer to peer networking and media streaming problems under the hood




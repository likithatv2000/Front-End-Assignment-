# Front-End-Assignment-
Front end
<!DOCTYPE html>
<html>

<head>
	<title>FILE UPLOAD DEMO</title>
</head>

<body>
	<h1>Single File Upload Demo</h1>

	<form action="/uploadProfilePicture"
	enctype="multipart/form-data" method="POST">
	
		<span>Upload Profile Picture:</span>
		<input type="file" name="mypic" required/> <br>
		<input type="submit" value="submit">
	</form>
</body>

</html>
let button;
let mic;
let soundRec;
let soundFile;


function setup() {
  mic = new p5.AudioIn();
  mic.start();
  soundRec = new p5.SoundRecorder();
  soundRec.setInput(mic)
  soundFile = new p5.SoundFile();

  button = createDiv("");
  button.position(100,100);
  button.size(100,100);
  button.style('background-color', 'grey');


  button.mouseClicked((mouseEvent)=>{
    console.log("recording....");
    soundRec.record(soundFile); // set up the soundfile to record and start recording

    let recordingTimer = setTimeout(()=>{ // setup a timeout for the recording, after the time below expires, do the tings inside the {}

      soundRec.stop(); // stop recording
      let soundBlob = soundFile.getBlob(); //get the recorded soundFile's blob & store it in a variable

      let formdata = new FormData() ; //create a from to of data to upload to the server
      formdata.append('soundBlob', soundBlob,  'myfiletosave.wav') ; // append the sound blob and the name of the file. third argument will show up on the server as req.file.originalname

          // Now we can send the blob to a server...
      var serverUrl = '/upload'; //we've made a POST endpoint on the server at /upload
      //build a HTTP POST request
      var httpRequestOptions = {
        method: 'POST',
        body: formdata , // with our form data packaged above
        headers: new Headers({
          'enctype': 'multipart/form-data' // the enctype is important to work with multer on the server
        })
      };
      // console.log(httpRequestOptions);
      // use p5 to make the POST request at our URL and with our options
      httpDo(
        serverUrl,
        httpRequestOptions,
        (successStatusCode)=>{ //if we were successful...
          console.log("uploaded recording successfully: " + successStatusCode)
        },
        (error)=>{console.error(error);}
      )
      console.log('recording stopped');

    },1000) //record for one second

  }) // close mouseClicked handler


} //close setup()

const express = require("express")
const path = require("path")
const multer = require("multer")
const app = express()
	
// View Engine Setup
app.set("views",path.join(__dirname,"views"))
app.set("view engine","ejs")
	
// var upload = multer({ dest: "Upload_folder_name" })
// If you do not want to use diskStorage then uncomment it
	
var storage = multer.diskStorage({
	destination: function (req, file, cb) {

		// Uploads is the Upload_folder_name
		cb(null, "uploads")
	},
	filename: function (req, file, cb) {
	cb(null, file.fieldname + "-" + Date.now()+".jpg")
	}
})
	
// Define the maximum size for uploading
// picture i.e. 1 MB. it is optional
const maxSize = 1 * 1000 * 1000;
	
var upload = multer({
	storage: storage,
	limits: { fileSize: maxSize },
	fileFilter: function (req, file, cb){
	
		// Set the filetypes, it is optional
		var filetypes = /jpeg|jpg|png/;
		var mimetype = filetypes.test(file.mimetype);

		var extname = filetypes.test(path.extname(
					file.originalname).toLowerCase());
		
		if (mimetype && extname) {
			return cb(null, true);
		}
	
		cb("Error: File upload only supports the "
				+ "following filetypes - " + filetypes);
	}

// mypic is the name of file attribute
}).single("mypic");	

app.get("/",function(req,res){
	res.render("Signup");
})
	
app.post("/uploadProfilePicture",function (req, res, next) {
		
	// Error MiddleWare for multer file upload, so if any
	// error occurs, the image would not be uploaded!
	upload(req,res,function(err) {

		if(err) {

			// ERROR occured (here it can be occured due
			// to uploading image of size greater than
			// 1MB or uploading different file type)
			res.send(err)
		}
		else {

			// SUCCESS, image successfully uploaded
			res.send("Success, Image uploaded!")
		}
	})
})
	
// Take any port number of your choice which
// is not taken by any other process
app.listen(8080,function(error) {
	if(error) throw error
		console.log("Server created Successfully on PORT 8080")
})

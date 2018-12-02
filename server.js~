var express = require('express');
var session = require('cookie-session');
var bodyParser = require('body-parser');
var app = express();
var fs = require('fs');
var formidable = require('formidable');
var MongoClient = require('mongodb').MongoClient;
var assert = require('assert');
var ObjectID = require('mongodb').ObjectID;

var mongourl = 'mongodb://g1192738:_Bblove630244@ds149672.mlab.com:49672/g1192738';

app.set('view engine', 'ejs');


var users = new Array(
    {userid: 'developer', password: 'developer'},
    {userid: 'guest', password: 'guest'}
);


app.use(session({
  name: 'session',
  keys: ['userid','authenticated']
}));
app.use(bodyParser.urlencoded({ extended: false }));
app.use(bodyParser.json());

app.get('/',function(req,res) {
    console.log(req.session);
    if (!req.session.authenticated) {
        res.redirect('/login');
    } else {
        res.status(200);
        
    }
});

app.get('/login',function(req,res){
    res.render('login');
});

app.post('/login',function(req,res) {
    for (var i=0; i<users.length; i++) {
        if (users[i].userid == req.body.userid &&
            users[i].password == req.body.password) {
            req.session.authenticated = true;
            req.session.userid = users[i].userid;
            console.log(req.session);
res.redirect('/display'); return;
        }
    }
    
});


app.get('/display', function(req,res) {
	if(req.session.authenticated == true){
  MongoClient.connect(mongourl, function(err,db) {
    try {
      assert.equal(err,null);
    } catch (err) {
      res.set({"Content-Type":"text/plain"});
      res.status(500).end("MongoClient connect() failed!");
    }
    console.log('Connected to MongoDB');
    findPhoto(db,{},function(restaurants) {
      db.close();
      console.log('Disconnected MongoDB');
      res.render("list.ejs",{restaurants:restaurants});
    });
});
}
});


app.get('/logout', function(req,res){
req.session = null;
console.log('logout successfully!');
res.redirect('/');
});

app.get('/create',function(req,res){
	
    res.render('create');

});

app.post('/create', function(req,res){
var form = new formidable.IncomingForm();
form.parse(req, function (err, fields, files) {
var filename = files.filetoupload.path;
if (files.filetoupload.type) {
	var mimetype = files.filetoupload.type;
}
fs.readFile(filename, function(err,data) {
	MongoClient.connect(mongourl, function(err,db){
var new_r = {};    // document to be inserted
    new_r['restaurant_id'] = fields.restaurant_id;
    new_r['name'] = fields.name;
    new_r['borough'] = fields.borough;
    new_r['cuisine'] = fields.cuisine;
    if(data){
        new_r['photo'] = new Buffer(data).toString('base64');
        new_r['photo_mimetype'] = files.filetoupload.type;
    }
        new_r['address'] = {street: fields.street, building: fields.building, zipcode: fields.zipcode, coord: [fields.lat, fields.lon]};
    new_r['grades'] = [{user: fields.user, score: fields.score}];
    new_r['owner'] = req.session.userid;
	
      
		assert.equal(err,null);
		console.log('Connected to MongoDB\n');	
    		insertDocument(db,new_r,function(result){
        	db.close();
        	req.session.userid = fields.userid;
	});
      });
});
        	res.redirect('/display'); return;
    });

});

app.get('/edit',function(req,res){
	console.log(JSON.stringify(req.query));
    res.render('edit.ejs', {_id:req.query._id});
});


app.post('/edit', function(req,res){
console.log('About to update ');
console.log("Receive data:" + JSON.stringify(req.body));
	MongoClient.connect(mongourl,function(err,db) {
		assert.equal(err,null);
		console.log('Connected to MongoDB\n');
		var criteria = {};
		criteria['restaurant_id'] = req.body.restaurant_id;
		var newValues = {};
		
		
		var address = {};
		for (var key in req.body) {
			if (key == "_id") {
				continue;
			}
			switch(key) {
				case 'restaurant_id':
					newValues[key] = req.body[key];
					break;
				case 'name':
					newValues[key] = req.body[key];
					break;
				case 'borough':
					newValues[key] = req.body[key];
					break;
				case 'cuisine': 
					newValues[key] = req.body[key];
					break;
				case 'street':
				case 'building':
				case 'zipcode':
				case 'lat':
				case 'lon':
					address[key] = req.body[key];
					break;
				case 'user':
				case 'score':
					newValues[key] = req.body[key];
					break;
				case 'owner':
					newValues[key] = req.body[key]
					break;
				case 'filetoupload':
					newValues[key] = req.body[key];
					break;
				default:
					newValues[key] = req.body[key];	
					break;
			}
		}
		if (address.lenght > 0) {
			newValues['address'] = address;
		}
		if (Object.keys(newValues).length<1)
		{
			return;
		}
		console.log('Preparing update: ' + JSON.stringify(newValues));
		console.log("Criteria: "+ JSON.stringify(criteria));
		updateDocument(db,criteria,newValues,function(result) {
			db.close();			
		});
	});
	res.redirect('/display'); return;
});


app.get('/show', function(req,res) {
  MongoClient.connect(mongourl, function(err,db) {
    try {
      assert.equal(err,null);
    } catch (err) {
      res.set({"Content-Type":"text/plain"});
      res.status(500).end("MongoClient connect() failed!");
    }      
    console.log('Connected to MongoDB');
    var criteria = {};
    criteria['_id'] = ObjectID(req.query._id);
    findPhoto(db,criteria, function(photo) {
      db.close();
      console.log('Disconnected MongoDB');
      console.log('Photo returned = ' + photo.length);
      var image = new Buffer(photo[0].photo,'base64');        
      var contentType = {};
      contentType['Content-Type'] = photo[0].photo_mimetype;
      console.log(contentType['Content-Type']);
      if (contentType['Content-Type'] == "image/jpeg") {
        res.render('photo.ejs',{photo:photo});
      } else {
        res.set({"Content-Type":"text/plain"});
        res.status(500).end("Not JPEG format!!!");  
      }
    });
  });
});

app.get('*', function(req,res) {
  res.redirect('/login');
});




function insertDocument(db,r,callback) {
  db.collection('restaurants').insertOne(r,function(err,result) {
    assert.equal(err,null);
    console.log("insert was successful!");
	console.log(result);
    callback(result);
  });
}

function updateDocument(db,criteria,newValues,callback) {
	db.collection('restaurants').updateOne(
		criteria,{$set: newValues},function(err,result) {
			assert.equal(err,null);
			console.log("update was successfully");
			console.log(result);
			callback(result);
	});
}

function findPhoto(db,criteria,callback) {
  var cursor = db.collection("restaurants").find(criteria);
  var photos = [];
  cursor.each(function(err,doc) {
    assert.equal(err,null);
    if (doc != null) {
      photos.push(doc);
    } else {
      callback(photos);
    }
  });
}



app.listen(process.env.PORT || 8099);


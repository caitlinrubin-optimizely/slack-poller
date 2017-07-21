// #!/usr/bin/env node

//C03DPN3QY -- engineering
var rp = require('request-promise')
var slack_token = process.env.SLACK_TOKEN;
var slack_channel = process.argv[2] || 'C03DPN3QY'
var mongoose = require('mongoose');
var Schema = mongoose.Schema
mongoose.connect(process.env.MONGO_URI);

var isQuestion = function(text) {
  return text.includes('?');
}

var lastTimeSentSchema = mongoose.Schema({
    value: Number
});

var TimeSent = mongoose.model('TimeSent', lastTimeSentSchema);

var questionSchema = mongoose.Schema({
    content: String,
    answers: [String]
});

questionSchema.index({'$**': 'text'});
var Question = mongoose.model('Question', questionSchema);

var last_time_sent;
var newest_last_time_sent;

//get last time sent
TimeSent.findOne()
				.then(function(timeSent) {
					last_time_sent = timeSent.value
					var options = {
					  method: 'POST',
					  uri: 'https://slack.com/api/channels.history',
					  qs: {
					    'token': slack_token,
					    'channel': slack_channel,
							'oldest': last_time_sent
					  },
					  json: true // Automatically stringifies the body to JSON
					}
					return rp(options);
				})
				.then(function(resp) {
					//bypass if no messages have been sent since the last time we polled
					if (resp.messages.length == 0) {
						return;
					}
					console.log("resp.messages: ", resp.messages)
					var messages = resp.messages;
					newest_last_time_sent = messages[0].ts;
					var texts = [];
					for (var i = messages.length-1; i >= 0; i--) {
						if (messages[i].text != '') {
							texts.push(messages[i].text);
						}
					}

					var found_question = false;
					var question_answers = [];
					var all_questions = [];
					for (var i = 0; i < texts.length; i++) {
						if (isQuestion(texts[i])) {
							found_question = true;
							if(question_answers.length > 0) {
								all_questions.push(question_answers);
							}
							question_answers = []
						}

						if (found_question) {
							question_answers.push(texts[i])
						}
					}

					if (question_answers.length > 0) {
						all_questions.push(question_answers)
					}

					for (var i = 0; i < all_questions.length; i++) {
						var content = all_questions[i];
						var question = content[0];
						var answers = content.slice(1);
						var answers_objs = []

						var q = new Question({
							content: question,
							answers: [answers]
						}).save()
					}
					return TimeSent.findOneAndUpdate({}, {$set:{value:newest_last_time_sent}});
				})
				.then(function(doc){
					console.log("SUCCESS: ", doc)
					process.exit();
				})
				.catch(function(err) {
					console.log("ERR:", err)
					process.exit();
				})

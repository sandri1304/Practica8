Plataphorma
===========

Meteor pet project created to teach my students the following Meteor functionality: 

* Collection's publish/subscribe 
* Deps.autorun 
* Meteor.methods/call 
* Integration of non-Meteor code in compatibility folder (HTML5 games Alien Invasion and Froot Wars)
* Usage of allow to control client access to collections

![ScreenShot](/screenshot.png)


Plataphorma offers the possibility to run 2 different HTML5 games: Alien Invasion and Froot Wars. 

On the right side of the screen the best players of each game are shown, updated in real time each time a signed in player finishes a game. If no game is selected, the best players overall are shown.

On the left side of the screen a chatroom for the current game is available. Only signed in users can post messages. If no game is selected a general chatroom is shown.

The original code of the two HTML5 games integrated in this project is available here:
* Alien Invasion: https://github.com/cykod/AlienInvasion
* Froot Wars: http://www.wrox.com/WileyCDA/WroxTitle/Professional-HTML5-Mobile-Game-Development.productCd-1118301323,descCd-DOWNLOAD.html

Bootstrap style (file bootstrap.min.css) provided by http://bootswatch.com


Running the project
-------------------

A live version of this code is running here: http://plataphorma.meteor.com

To run the project locally, clone the repo and run ```meteor``` inside it. You can see in .meteor/packages that this Meteor project uses these packages:
* ```meteor remove autopublish```
* ```meteor remove insecure```
* ```meteor add bootstrap```
* ```meteor add accounts-ui```
* ```meteor add accounts-password```




Respuestas a las preguntas de clase:

1) Click en el botón de juego 
	En el fichero client.js se capturan los eventos cuando se hace click en uno de los juegos con el siguiente código:
	Template.choose_game.events = {
		'click #AlienInvasion': function () {
		$('#gamecontainer').hide();
		$('#container').show();
		var game = Games.findOne({name:"AlienInvasion"});
		Session.set("current_game", game._id);
		},
		'click #FrootWars': function () {
		$('#container').hide();
		$('#gamecontainer').show();
		var game = Games.findOne({name:"FrootWars"});
		Session.set("current_game", game._id);
		},
		'click #none': function () {
		$('#container').hide();
		$('#gamecontainer').hide();
		Session.set("current_game", "none");
		}
	}
	
		Lo primero que se hace cuando se hace click en el boton de juego se ocultan el resto de los juegos y aparece el div del juego en el 		que has hecho el click, es decir, aparece el canvas del juego seleccionado.
2) Se escribe mensaje chat sin estar autenticado
 
	No se permite escribir un mensaje sin estar autenticado, mostrando un error 
	
	Esto se hace con el siguiente código que se encuentra en el fichero client.js:
	
	Template.input.events = {
    'keydown input#message' : function (event) {
	if (event.which == 13) { 
	    if (Meteor.userId()){
		var user_id = Meteor.user()._id;	    
		var message = $('#message');
		if (message.value != '') {
		    Messages.insert({
			user_id: user_id,
			message: message.val(),
			time: Date.now(),
			game_id: Session.get("current_game")
		    });
		    message.val('')
		}
	    }
	    else {
		$("#login-error").show();
	    }
	}
    }
	}

3) se escribe mensaje chat estando autenticado. 

	Te permite escribir un mensaje, se realiza con el siguiente código que se encuentra el fichero client.js:
	
	Template.input.events = {
    'keydown input#message' : function (event) {
	if (event.which == 13) { 
	    if (Meteor.userId()){
		var user_id = Meteor.user()._id;	    
		var message = $('#message');
		if (message.value != '') {
		    Messages.insert({
			user_id: user_id,
			message: message.val(),
			time: Date.now(),
			game_id: Session.get("current_game")
		    });
		    message.val('')
		}
	    }
	    else {
		$("#login-error").show();
	    }
	}
    }
	}
	
	Lo primero que se hace es capturar el evento cuando se pulsa la tecla enter, se comprueba si el usuario está autenticado y al estarlo, 		se guarda el identificador del usuario y el mensaje, si el mensaje no es vacio lo inserta en la coleccion de mensajes con el nombre 	del usuario que lo ha creado, con el valor del mensaje, la fecha en la que lo escribio y  el identificador del chat del juego en el 	que se ha escrito, finalmente vacía el input donde se escribe. 

4) Se termina partida con puntuacion mas alta sin estar autenticado. 
	Sin estar autenticado, la plataforma te permite jugar a cualquier juego pero no guarda la puntuacion que consigues este debido a que 		cuando wingame en el game.js o en el losegame.js:
	
	var winGame = function() {
    Meteor.call("matchFinish", Session.get("current_game"), Game.points);
    Game.setBoard(3,new TitleScreen("You win!", 
                                    "Press fire to play again",
                                    playGame));
	};
	
	var loseGame = function() {
    Meteor.call("matchFinish", Session.get("current_game"), Game.points);
    Game.setBoard(3,new TitleScreen("You lose!", 
                                    "Press fire to play again",
                                    playGame));
	};
	
	llama al metodo matchFinish se comprueba si esta autentificado como no lo esta no se guarda nada en la base de datos, coleccion de 		las partidas. 
	
		Meteor.methods({
    matchFinish: function (game, points) {
	// Don't insert in the Matches collection a match if the user
	// has not signed in
	if (this.userId)
	    Matches.insert ({user_id: this.userId, 
			     time_end: Date.now(),
			     points: points,
			     game_id: game
			    });
    }
	});
	
5) Se termina partida con puntuacion mas alta sin estar autenticado.
	
 	Este código se encuentra en el game.js.
	var winGame = function() {
    Meteor.call("matchFinish", Session.get("current_game"), Game.points);
    Game.setBoard(3,new TitleScreen("You win!", 
                                    "Press fire to play again",
                                    playGame));
	};
	
	var loseGame = function() {
    Meteor.call("matchFinish", Session.get("current_game"), Game.points);
    Game.setBoard(3,new TitleScreen("You lose!", 
                                    "Press fire to play again",
                                    playGame));
	};
	
	con el codigo anterior cuando se termina la partida se comprueba si un jugador gana  o pierde con los códigos anteriores, estas 	    funciones van a llamar al metodo matchFinish que es el siguiente código y aparece en el server.js se llama al siguiente método. 

	
	Meteor.methods({
    matchFinish: function (game, points) {
	// Don't insert in the Matches collection a match if the user
	// has not signed in
	if (this.userId)
	    Matches.insert ({user_id: this.userId, 
			     time_end: Date.now(),
			     points: points,
			     game_id: game
			    });
    }
	});
	
	Lo primero que comprueba si el usuario que esta jugando esta autentificado, como en este caso si lo esta inserta en la coleccion de 	partidas en el identificador del usuario que esta jugando, la fecha de finalizacion, la puntuacion y el identificador del juego en que 		el ha conseguido los juegos. 

6)¿Que sale en la consola cuando entras?
 "current user: null"
 "current user: ZJhyte6TNZiwjWjSL"
 
 var currentUser = null;
	Deps.autorun(function(){
		console.log("current user: " + currentUser);
		currentUser = Meteor.userId();
		console.log("current user: " + currentUser);
	});
	
	Lo unico que hace es hacer autorun mostrando lo que sale por pantalla, de ahi que lo primero que salga sea null porque la variable currentUser tiene ese valor, posteriormente se comprueba el id del usuario y lo muestra. 
 
7) ¿Qué sale en la consola cuando te sales?
	"current user: ZJhyte6TNZiwjWjSL"
	"current user: null"
	
	var currentUser = null;
	Deps.autorun(function(){
		console.log("current user: " + currentUser);
		currentUser = Meteor.userId();
		console.log("current user: " + currentUser);
	});
	
	en este caso, puesto que ahora esta autenticado te muestra el valor del id,  pone el valor de currentUser a null y cuando lo vuelve a comprobar este valor esta a null y es lo que te muestra. 
	
	

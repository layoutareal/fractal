
var params = {
	width: 800, //same as dom
	height: 800 //same as dom
};

var config = {
	treeStartPoint: {
		x: 0,
		y: -300
	},
	treeConfig:{
		angle: 20,
		levels: 8,
		initialLength: 120,
		lengthRatio: 0.86,
		lengthLeftRightRatio: 1
	}
};

class Tree{
	constructor(startPoint, config){
		this.startPoint = startPoint;
		this.config = config;
		this.lines = [];
	}
	//TODO: use setters
	setConfig(newConfig){
		this.config = newConfig;
	}

	calculate(){
		var initialFrom = toAngle(this.startPoint, 90, this.config.initialLength);
		this.lines = [];
		for (var i=0; i<=this.config.levels-1; i++){
			this.lines.push([]);
		}
    //TODO maybe have the line structure in place, and only update the coords might be faster
		this.lines[0].push({
			from: this.startPoint,
			to: initialFrom,
			level: 1
		});

    //recursive computation of line coordinates
		var doTreeR = (point, prevAngle, prevLength, lvl) => {
      //if we readched the maximul level, exit the branch
			if (lvl > this.config.levels) return;
			var len = prevLength * this.config.lengthRatio;
			var lenRight = len * this.config.lengthLeftRightRatio;
			//len = prevLength - 4;
			var leftAngle = prevAngle - this.config.angle;
			var rightAngle = prevAngle + this.config.angle;
			var leftTo = toAngle(point, leftAngle, len);
			var rightTo = toAngle(point, rightAngle, lenRight);

			//push left branch
			this.lines[lvl-1].push({
				from: point,
				to: leftTo,
				level: lvl
			});
			//push right branch
			this.lines[lvl-1].push({
				from: point,
				to: rightTo,
				level: lvl
			});

			doTreeR(leftTo, leftAngle, len, lvl+1);
			doTreeR(rightTo, rightAngle, len, lvl+1);
		};
    //start the recursive calculation
		doTreeR(initialFrom, 90, this.config.initialLength, 2);
	}
	draw(ctx){
		//draw based on previous line calculation
		ctx.clearRect(0,0, params.width, params.height);

		ctx.shadowColor = "red"; //a bit of red glow
		ctx.shadowBlur = 4;
		ctx.lineCap = 'round';
		ctx.globalCompositeOperation = "lighter";

		for (var i = this.lines.length - 1; i >= 0; i--){
			var lvl = this.lines[i];
			var lvlLength = lvl.length;
			var l =  (i+1)/this.config.levels * 40 + 30;
			ctx.lineWidth = (this.config.levels - i) / this.config.levels * 5;

			lvl.forEach((line, idx)=>{
				idx = idx + 1;
				//color, cycle through 0->360 based on the index in the level
				var c = 270 - idx / lvlLength * 270;

				var from = translateToCanvas(line.from.x, line.from.y);
				var to = translateToCanvas(line.to.x, line.to.y);

				ctx.beginPath();

				ctx.strokeStyle = `hsl(${c}, 90%, ${l}%)`;

				ctx.moveTo(from.i, from.j);
				ctx.lineTo(to.i, to.j);

				ctx.stroke();
			});
		}
	}

}

var randomizer = {

	prevTime: 0,
	diff: 0,
	pos: 0,
	dir: 1,
	paused: true,
	tick: function(){
		if (this.paused) return;
		var now = new Date();
		var nowMS = now.getTime();
		var nowS = Math.floor(nowMS / 1000);

		if (this.prevTime === 0) {
			this.diff = 0;
			this.prevTime = nowMS;
			return;
		} else {
			this.diff = nowMS - this.prevTime;
		}
		this.prevTime = nowMS;

		var totalTime = 3000; //seconds
		var diff = this.diff / totalTime;
		var pos = this.pos + diff * this.dir;

		if (pos > 1){
			pos = 1 - pos/10;
			this.dir = -1;
		}
		if (pos < 0){
			pos = pos/10;
			this.dir = 1;
		}
		config.treeConfig.angle = pos * 70;
		this.pos = pos;
	},
	start: function(){
		this.paused = false;
	},
	stop: function(){
		this.prevTime = 0;
		this.diff = 0;
		this.pos = 0;
		this.dir = 1;
		this.paused = true;
	}
};

jQuery(function($){ //bootstrap when dom ready
	var $canvas = $('canvas');
	var canvas = $canvas.get(0);
	var ctx = canvas.getContext('2d');

	var stats = new Stats();
	stats.domElement.style.position = 'absolute';
	stats.domElement.style.right = '0px';
	stats.domElement.style.top = '0px';
	document.body.appendChild(stats.domElement);


	var tree = new Tree(config.treeStartPoint, config.treeConfig);

	var loop = ()=>{
		tree.setConfig(config.treeConfig);

		randomizer.tick();

		tree.calculate();
		tree.draw(ctx);
		requestAnimationFrame(loop);
		stats.update();
	};
	randomizer.start(); //start the animation by default
	loop();

	var runToId = null;
	$('canvas').on('mousemove', (ev) => {
    var $b = $('body');
	  var bodyW = 800;
  	var bodyH = 800;
		var x = ev.pageX;
		var y = ev.pageY;
    
		config.treeConfig.angle = y/bodyH *70;
		config.treeConfig.lengthLeftRightRatio = x/bodyW;
		randomizer.stop();
		clearTimeout(runToId);
		runToId = setTimeout(()=>randomizer.start(), 2000);
	});

});

/**
HELPER FUNCTIONS
**/
var toRad = (a) => Math.PI / 180 * a;

/**
 * toAngle({x:1, y:2}, 20, 30)
 * or
 * toAngle(1,2,20, 30);
 */
var toAngle = (x, y, a, len) => {
	if (typeof x === "object") {
		len = a;
		a = y;
		y = x.y;
		x = x.x;
	}
	var ret = {
		x: x + len * Math.cos(toRad(a)),
		y: y + len * Math.sin(toRad(a))
	};
	return ret;
};

var translateToCanvas = function (x, y) {
	var i;
	var j;

	i = params.width / 2 + x;
	j = params.height / 2 - y + 1;
	return {
		i: i,
		j: j
	};
};

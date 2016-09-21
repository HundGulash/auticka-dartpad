# auticka-dartpad

import 'dart:html';
import 'dart:math' as math;
import 'dart:collection';

var redCar = new ImageElement(src: "http://www.clker.com/cliparts/n/E/6/Q/E/m/red-car-top-view-th.png?1");
var yellowCar = new ImageElement(src: "http://s3.postimg.org/8kxz56kzj/car_icon_top_view_1474257.png?1");
var greenCar = new ImageElement(src:"http://www.clker.com/cliparts/o/h/i/f/Y/K/red-car-top-view-th.png?1");
const double dvePi = 2.0*math.PI;

Auto a,b,c,d,e;
var auta;

CanvasElement canvas;
CanvasRenderingContext2D ctx;
Keyboard keyboard = new Keyboard();

class Auto {
  double x;
  double y;
  ImageElement image;
  double orientation = 0.0; // orientation of the image itself
  double speed = 0.0;
  double angle;
  double step = math.PI/90.0;
  double minSteeringSpeed = 0.1;
  double accelerateDistance = 45.0;
  double decelarateDistance = 40.0;
  double maxSpeed = 5.0;
  double acceleration = 0.03;
  double scale = 0.2;
  Auto target;

  Auto(double x0, double y0, ImageElement img, double scale, double orientation) {
    x = x0;
    y = y0;
    speed = 0.0;
    angle = -math.PI/2;
    image = img;
    this.scale = scale;
    this.orientation = orientation;
  }

  void move() {
    x += math.cos(angle)*speed;
    y += math.sin(angle)*speed;
    speed = speed * 0.99;

    if (target != null)
      follow(target);
  }

  void follow(Auto target) {
		// nesledovat stred auta, ale bod kousek za autem
		double ax = target.x - math.cos(target.angle)*target.image.width*target.scale;
    double ay = target.y - math.sin(target.angle)*target.image.height*target.scale;
    followPoint(ax, ay);
  }
    
  void followPoint(double px, double py) {
    double dx = px - x;
    double dy = py - y;
    double distance = math.sqrt(dx*dx + dy*dy);

    if (distance > accelerateDistance)
      accelerate();
    if(distance < decelarateDistance)
      brake();

    if (dx == 0.0)
      return;

    double azimuth = math.atan(dy/dx);
    double dangle = angle - azimuth;
    
    if (dx < 0)
      dangle += math.PI;
    dangle = normalizeAngle(dangle);
    if (dangle <= math.PI)
      steerLeft();
    else
      steerRight();
  }

  void steerRight() {
    if (speed > minSteeringSpeed)
      angle += step;
    else if (speed < -minSteeringSpeed)
      angle -= step;

    angle = normalizeAngle(angle);
  }

  void steerLeft() {
    if (speed > minSteeringSpeed)
      angle -= step;
    else if (speed < -minSteeringSpeed)
      angle += step;

    angle = normalizeAngle(angle);
  }

  void accelerate() {
    if (speed < 0.0 && speed > -0.5) // pozpatku a pomalu?
      speed = 0.0; // stop
    else if (speed < maxSpeed)
      speed += acceleration;
  }

  void decelerate() {
    if (speed > 0 && speed < 1.0)
      speed = 0.0;
    else if (speed>=-2)
      speed -= acceleration;
  }

  void brake() {
    speed = speed * 0.94;
  }

  void draw(CanvasRenderingContext2D context) {
    context.translate(x,y);
    context.rotate(angle + orientation);
    context.scale(scale,scale);
    context.translate(-x,-y);
    context.drawImage(image, x - image.width/2, y - image.height/2);
  }

  double normalizeAngle(double angle) {
    while (angle > dvePi)
        angle -= dvePi;
    while(angle < 0.0)
      angle += dvePi;
    return angle;
  }
}

class Keyboard {
  HashMap<int, int> _keys = new HashMap<int, int>();

  Keyboard() {
    window.onKeyDown.listen((KeyboardEvent event) {
      if (!_keys.containsKey(event.keyCode)) {
        _keys[event.keyCode] = event.timeStamp;
      }
    });

    window.onKeyUp.listen((KeyboardEvent event) {
      _keys.remove(event.keyCode);
    });
  }

  bool isPressed(int keyCode) => _keys.containsKey(keyCode);
}

update(num time) {
  show();
  run();
}

show() {
  ctx.setFillColorRgb(40,40,40);
  ctx.fillRect(0,0, ctx.canvas.width, ctx.canvas.height);

  for(var auto in auta) {
    ctx.save();
    auto.draw(ctx);
    ctx.restore();
  }

  for(var auto in auta) {
    auto.move();
  }

  handleKeys();
}

void init() {
	canvas = querySelector("#canvas");
  ctx = canvas.getContext("2d");

  double dx = 23.0;
  a = new Auto(canvas.width/4*3, canvas.height/4*3, yellowCar, 0.2, math.PI/2.0);
  b = new Auto(a.x - dx, a.y, redCar, 0.3, 0.0);
  c = new Auto(b.x - dx, a.y, greenCar, 0.3, 0.0);
  d = new Auto(c.x - dx, a.y, redCar, 0.3, 0.0);
  e = new Auto(d.x - dx, a.y, greenCar, 0.3, 0.0);


  b.target = a;
  b.maxSpeed = 2.0;
  b.acceleration = 0.02;
  c.target = b;
  c.maxSpeed = 1.8;
  c.acceleration = 0.02;
  d.target = c;
  e.target = d;

  auta = [a,b,c,d,e];
}

run() {
  window.animationFrame.then(update);
}

handleKeys() {
  if (keyboard.isPressed(KeyCode.UP))
    a.accelerate();
  if (keyboard.isPressed(KeyCode.DOWN))
    a.decelerate();
  if (keyboard.isPressed(KeyCode.RIGHT))
    a.steerRight();
  if (keyboard.isPressed(KeyCode.LEFT))
    a.steerLeft();
  if (keyboard.isPressed(KeyCode.H))
    a.followPoint(canvas.width/2.0, canvas.height/2.0);
  if (keyboard.isPressed(KeyCode.SPACE))
    a.brake();
}

main() {
  init();
  run();
}

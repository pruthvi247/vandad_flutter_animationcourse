[source](https://www.youtube.com/watch?v=z6KweorVrNk&list=PL6yRaaP0WPkW3kwAerPeRqGBvJfO8O4S7&index=3)

[source](https://medium.com/flutter/animation-deep-dive-39d3ffea111f)

Animations in Flutter are just a way to rebuild parts of your widget tree on every frame. There is no special case. Flutter is fast enough to do that.

The individual pictures are called frames in cinema— and because digital screens work similarly— they’re called frames here too. Cinema normally shows 24 frames per second. Modern digital devices show 60 to 120 frames per second.

**AnimationController**

AnimationController is what you normally use to play, pause, reverse, and stop animations. Instead of pure “tick” events, AnimationController tells us at which point of the animation we are, at any time. For example, are we halfway there? Are we 99% there? Have we completed the animation?

Normally, you take the AnimationController, maybe transform it with a Curve, put it through a Tween, and use it in one of the handy widgets like FadeTransition or TweenAnimationBuilder.

[source](https://docs.flutter.dev/ui/animations/tutorial)

AnimationController is a special Animation object that generates a new value whenever the hardware is ready for a new frame. By default, an AnimationController linearly produces the numbers from 0.0 to 1.0 during a given duration. For example, this code creates an Animation object, but does not start it running:

```dart
controller =
    AnimationController(duration: const Duration(seconds: 2), vsync: this);
```

AnimationController derives from Animation<double>, so it can be used wherever an Animation object is needed. However, the AnimationController has additional methods to control the animation. For example, you start an animation with the .forward() method. The generation of numbers is tied to the screen refresh, so typically 60 numbers are generated per second. After each number is generated, each Animation object calls the attached Listener objects. To create a custom display list for each child, see RepaintBoundary.

When creating an AnimationController, you pass it a vsync argument. The presence of vsync prevents offscreen animations from consuming unnecessary resources. You can use your stateful object as the vsync by adding SingleTickerProviderStateMixin to the class definition. You can see an example

```dart
import 'package:flutter/material.dart';

void main() => runApp(const LogoApp());

class LogoApp extends StatefulWidget {
  const LogoApp({super.key});

  @override
  State<LogoApp> createState() => _LogoAppState();
}

class _LogoAppState extends State<LogoApp> with SingleTickerProviderStateMixin {
  late Animation<double> animation;
  late AnimationController controller;

  @override
  void initState() {
    super.initState();
    controller =
        AnimationController(duration: const Duration(seconds: 2), vsync: this);
    // #docregion addListener
    animation = Tween<double>(begin: 0, end: 300).animate(controller)
      ..addListener(() {
        // #enddocregion addListener
        setState(() {
          // The state that has changed here is the animation object’s value.
        });
        // #docregion addListener
      });
    // #enddocregion addListener
    controller.forward();
  }

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Container(
        margin: const EdgeInsets.symmetric(vertical: 10),
        height: animation.value,
        width: animation.value,
        child: const FlutterLogo(),
      ),
    );
  }

  @override
  void dispose() {
    controller.dispose();
    super.dispose();
  }
}
```

Tween
By default, the AnimationController object ranges from 0.0 to 1.0. If you need a different range or a different data type, you can use a Tween to configure an animation to interpolate to a different range or data type. For example, the following Tween goes from -200.0 to 0.0:

```dart
tween = Tween<double>(begin: -200, end: 0);
```

A Tween is a stateless object that takes only begin and end. The sole job of a Tween is to define a mapping from an input range to an output range. The input range is commonly 0.0 to 1.0, but that’s not a requirement.

A Tween inherits from Animatable<T>, not from Animation<T>. An Animatable, like Animation, doesn’t have to output double. For example, ColorTween specifies a progression between two colors.

```dart
colorTween = ColorTween(begin: Colors.transparent, end: Colors.black54);
```

A Tween object doesn’t store any state. Instead, it provides the evaluate(Animation<double> animation) method that uses the transform function to map the current value of the animation (between 0.0 and 1.0), to the actual animation value.

The current value of the Animation object can be found in the .value method. The evaluate function also performs some housekeeping, such as ensuring that begin and end are returned when the animation values are 0.0 and 1.0, respectively.

## Tween.animate

To use a Tween object, call animate() on the Tween, passing in the controller object. For example, the following code generates the integer values from 0 to 255 over the course of 500 ms.

```dart
AnimationController controller = AnimationController(
    duration: const Duration(milliseconds: 500), vsync: this);
Animation<int> alpha = IntTween(begin: 0, end: 255).animate(controller);
```

The following example shows a controller, a curve, and a Tween:

```dart
AnimationController controller = AnimationController(
    duration: const Duration(milliseconds: 500), vsync: this);
final Animation<double> curve =
    CurvedAnimation(parent: controller, curve: Curves.easeOut);
Animation<int> alpha = IntTween(begin: 0, end: 255).animate(curve);
```

To render with an Animation object, store the Animation object as a member of your widget, then use its value to decide how to draw.

Consider the following app that draws the Flutter logo without animation:

The following shows the same code modified to animate the logo to grow from nothing to full size. When defining an AnimationController, you must pass in a vsync object. The vsync parameter is described in the AnimationController section.

The changes from the non-animated example are highlighted:

```dart
 State<LogoApp> createState() => _LogoAppState();
                                 }
                    -            class _LogoAppState extends State<LogoApp> {
                    +            class _LogoAppState extends State<LogoApp> with SingleTickerProviderStateMixin {
                    +              late Animation<double> animation;
                    +              late AnimationController controller;
                    +
                    +              @override
                    +              void initState() {
                    +                super.initState();
                    +                controller =
                    +                    AnimationController(duration: const Duration(seconds: 2), vsync: this);
                    +                animation = Tween<double>(begin: 0, end: 300).animate(controller)
                    +                  ..addListener(() {
                    +                    setState(() {
                    +                      // The state that has changed here is the animation object’s value.
                    +                    });
                    +                  });
                    +                controller.forward();
                    +              }
                    +
                                   @override
                                   Widget build(BuildContext context) {
                                     return Center(
                                       child: Container(
                                         margin: const EdgeInsets.symmetric(vertical: 10),
                    -                    height: 300,
                    -                    width: 300,
                    +                    height: animation.value,
                    +                    width: animation.value,
                                         child: const FlutterLogo(),
                                       ),
                                     );
                                   }
                    +
                    +              @override
                    +              void dispose() {
                    +                controller.dispose();
                    +                super.dispose();
                    +              }
                                 }
```

The addListener() function calls setState(), so every time the Animation generates a new number, the current frame is marked dirty, which forces build() to be called again. In build(), the container changes size because its height and width now use animation.value instead of a hardcoded value. Dispose of the controller when the State object is discarded to prevent memory leaks.

# Simplifying with Animated­Widget [](https://docs.flutter.dev/ui/animations/tutorial#simplifying-with-animatedwidget)

> What's the point?
> How to use the AnimatedWidget helper class (instead of addListener() and setState()) to create a widget that animates.
> Use AnimatedWidget to create a widget that performs a reusable animation. To separate the transition from the widget, use an AnimatedBuilder, as shown in the Refactoring with AnimatedBuilder section.
> Examples of AnimatedWidgets in the Flutter API: AnimatedBuilder, AnimatedModalBarrier, DecoratedBoxTransition, FadeTransition, PositionedTransition, RelativePositionedTransition, RotationTransition, ScaleTransition, SizeTransition, SlideTransition.

The AnimatedWidget base class allows you to separate out the core widget code from the animation code. AnimatedWidget doesn’t need to maintain a State object to hold the animation. Add the following AnimatedLogo class:
`Below code has git diff of both animatedwidget and setstate`

```dart
import 'package:flutter/material.dart';
                                 void main() => runApp(const LogoApp());
                    +            class AnimatedLogo extends AnimatedWidget {
                    +              const AnimatedLogo({super.key, required Animation<double> animation})
                    +                  : super(listenable: animation);
                    +
                    +              @override
                    +              Widget build(BuildContext context) {
                    +                final animation = listenable as Animation<double>;
                    +                return Center(
                    +                  child: Container(
                    +                    margin: const EdgeInsets.symmetric(vertical: 10),
                    +                    height: animation.value,
                    +                    width: animation.value,
                    +                    child: const FlutterLogo(),
                    +                  ),
                    +                );
                    +              }
                    +            }
                    +
                                 class LogoApp extends StatefulWidget {
                                   const LogoApp({super.key});
                                   @override
                                   State<LogoApp> createState() => _LogoAppState();
                                 }
        @@ -15,32 +33,18 @@
                                   @override
                                   void initState() {
                                     super.initState();
                                     controller =
                                         AnimationController(duration: const Duration(seconds: 2), vsync: this);
                    -                animation = Tween<double>(begin: 0, end: 300).animate(controller)
                    -                  ..addListener(() {
                    -                    setState(() {
                    -                      // The state that has changed here is the animation object’s value.
                    -                    });
                    -                  });
                    +                animation = Tween<double>(begin: 0, end: 300).animate(controller);
                                     controller.forward();
                                   }
                                   @override
                    -              Widget build(BuildContext context) {
                    -                return Center(
                    -                  child: Container(
                    -                    margin: const EdgeInsets.symmetric(vertical: 10),
                    -                    height: animation.value,
                    -                    width: animation.value,
                    -                    child: const FlutterLogo(),
                    -                  ),
                    -                );
                    -              }
                    +              Widget build(BuildContext context) => AnimatedLogo(animation: animation);
                                   @override
                                   void dispose() {
                                     controller.dispose();
                                     super.dispose();
                                   }
```

AnimatedLogo uses the current value of the animation when drawing itself.

The LogoApp still manages the AnimationController and the Tween, and it passes the Animation object to AnimatedLogo:

> Next, use addStatusListener() to reverse the animation at the beginning or the end. This creates a “breathing” effect:
> `{animate2 → animate3}/lib/main.dart`

```dart
void initState() {
                                     super.initState();
                                     controller =
                                         AnimationController(duration: const Duration(seconds: 2), vsync: this);
                    -                animation = Tween<double>(begin: 0, end: 300).animate(controller);
                    +                animation = Tween<double>(begin: 0, end: 300).animate(controller)
                    +                  ..addStatusListener((status) {
                    +                    if (status == AnimationStatus.completed) {
                    +                      controller.reverse();
                    +                    } else if (status == AnimationStatus.dismissed) {
                    +                      controller.forward();
                    +                    }
                    +                  })
                    +                  ..addStatusListener((status) => print('$status'));
                                     controller.forward();
                                   }
```

### Refactoring with AnimatedBuilder

> What's the point?
> An AnimatedBuilder understands how to render the transition.
> An AnimatedBuilder doesn’t know how to render the widget, nor does it manage the Animation object.
> Use AnimatedBuilder to describe an animation as part of a build method for another widget. If you simply want to define a widget with a reusable animation, use an AnimatedWidget, as shown in the Simplifying with AnimatedWidget section.
> Examples of AnimatedBuilders in the Flutter API: BottomSheet, ExpansionTile, PopupMenu, ProgressIndicator, RefreshIndicator, Scaffold, SnackBar, TabBar, TextField.

One problem with the code in the animate3 example, is that changing the animation required changing the widget that renders the logo. A better solution is to separate responsibilities into different classes:

- Render the logo
- Define the Animation object
- Render the transition

You can accomplish this separation with the help of the AnimatedBuilder class. An AnimatedBuilder is a separate class in the render tree. Like AnimatedWidget, AnimatedBuilder automatically listens to notifications from the Animation object, and marks the widget tree dirty as necessary, so you don’t need to call addListener().

The widget tree for the [animate4](https://github.com/flutter/website/tree/main/examples/animation/animate4) example looks like this:

AnimatedBuilder-WidgetTree.png
Starting from the bottom of the widget tree, the code for rendering the logo is straightforward:

```dart
class LogoWidget extends StatelessWidget {
  const LogoWidget({super.key});

  // Leave out the height and width so it fills the animating parent
  @override
  Widget build(BuildContext context) {
    return Container(
      margin: const EdgeInsets.symmetric(vertical: 10),
      child: const FlutterLogo(),
    );
  }
}
```

The middle three blocks in the diagram are all created in the build() method in GrowTransition, shown below. The GrowTransition widget itself is stateless and holds the set of final variables necessary to define the transition animation. The build() function creates and returns the AnimatedBuilder, which takes the (Anonymous builder) method and the LogoWidget object as parameters. The work of rendering the transition actually happens in the (Anonymous builder) method, which creates a Container of the appropriate size to force the LogoWidget to shrink to fit.

One tricky point in the code below is that the child looks like it’s specified twice. What’s happening is that the outer reference of child is passed to AnimatedBuilder, which passes it to the anonymous closure, which then uses that object as its child. The net result is that the AnimatedBuilder is inserted in between the two widgets in the render tree.

```dart
class GrowTransition extends StatelessWidget {
  const GrowTransition(
      {required this.child, required this.animation, super.key});

  final Widget child;
  final Animation<double> animation;

  @override
  Widget build(BuildContext context) {
    return Center(
      child: AnimatedBuilder(
        animation: animation,
        builder: (context, child) {
          return SizedBox(
            height: animation.value,
            width: animation.value,
            child: child,
          );
        },
        child: child,
      ),
    );
  }
}
```

Finally, the code to initialize the animation looks very similar to the animate2 example. The initState() method creates an AnimationController and a Tween, then binds them with animate(). The magic happens in the build() method, which returns a GrowTransition object with a LogoWidget as a child, and an animation object to drive the transition. These are the three elements listed in the bullet points above.
`{animate2 → animate4}/lib/main.dart`

```dart
import 'package:flutter/material.dart';
                                 void main() => runApp(const LogoApp());
                    -            class AnimatedLogo extends AnimatedWidget {
                    -              const AnimatedLogo({super.key, required Animation<double> animation})
                    -                  : super(listenable: animation);
                    +            class LogoWidget extends StatelessWidget {
                    +              const LogoWidget({super.key});
                    +
                    +              // Leave out the height and width so it fills the animating parent
                    +              @override
                    +              Widget build(BuildContext context) {
                    +                return Container(
                    +                  margin: const EdgeInsets.symmetric(vertical: 10),
                    +                  child: const FlutterLogo(),
                    +                );
                    +              }
                    +            }
                    +
                    +            class GrowTransition extends StatelessWidget {
                    +              const GrowTransition(
                    +                  {required this.child, required this.animation, super.key});
                    +
                    +              final Widget child;
                    +              final Animation<double> animation;
                                   @override
                                   Widget build(BuildContext context) {
                    -                final animation = listenable as Animation<double>;
                                     return Center(
                    -                  child: Container(
                    -                    margin: const EdgeInsets.symmetric(vertical: 10),
                    -                    height: animation.value,
                    -                    width: animation.value,
                    -                    child: const FlutterLogo(),
                    +                  child: AnimatedBuilder(
                    +                    animation: animation,
                    +                    builder: (context, child) {
                    +                      return SizedBox(
                    +                        height: animation.value,
                    +                        width: animation.value,
                    +                        child: child,
                    +                      );
                    +                    },
                    +                    child: child,
                                       ),
                                     );
                                   }
                                 }
                                 class LogoApp extends StatefulWidget {
                                   const LogoApp({super.key});
                                   @override
                                   State<LogoApp> createState() => _LogoAppState();
        @@ -34,18 +54,23 @@
                                   @override
                                   void initState() {
                                     super.initState();
                                     controller =
                                         AnimationController(duration: const Duration(seconds: 2), vsync: this);
                                     animation = Tween<double>(begin: 0, end: 300).animate(controller);
                                     controller.forward();
                                   }
                                   @override
                    -              Widget build(BuildContext context) => AnimatedLogo(animation: animation);
                    +              Widget build(BuildContext context) {
                    +                return GrowTransition(
                    +                  animation: animation,
                    +                  child: const LogoWidget(),
                    +                );
                    +              }
                                   @override
                                   void dispose() {
                                     controller.dispose();
                                     super.dispose();
                                   }
                                 }
```

## Simultaneous animations

> Note: This example shows how to use multiple tweens on the same animation controller, where each tween manages a different effect in the animation. It is for illustrative purposes only. If you were tweening opacity and size in production code, you’d probably use FadeTransition and SizeTransition instead.

Each tween manages an aspect of the animation. For example:

```dart
controller =
    AnimationController(duration: const Duration(seconds: 2), vsync: this);
sizeAnimation = Tween<double>(begin: 0, end: 300).animate(controller);
opacityAnimation = Tween<double>(begin: 0.1, end: 1).animate(controller);
```

You can get the size with sizeAnimation.value and the opacity with opacityAnimation.value, but the constructor for AnimatedWidget only takes a single Animation object. To solve this problem, the example creates its own Tween objects and explicitly calculates the values.

Change AnimatedLogo to encapsulate its own Tween objects, and its build() method calls Tween.evaluate() on the parent’s animation object to calculate the required size and opacity values. The following code shows the changes with highlights:
[github](https://github.com/flutter/website/blob/main/examples/animation/animate5/lib/main.dart)

```dart
// #docregion ShakeCurve
import 'dart:math';

// #enddocregion ShakeCurve
import 'package:flutter/material.dart';

void main() => runApp(const LogoApp());

// #docregion diff
class AnimatedLogo extends AnimatedWidget {
  const AnimatedLogo({super.key, required Animation<double> animation})
      : super(listenable: animation);

  // Make the Tweens static because they don't change.
  static final _opacityTween = Tween<double>(begin: 0.1, end: 1);
  static final _sizeTween = Tween<double>(begin: 0, end: 300);

  @override
  Widget build(BuildContext context) {
    final animation = listenable as Animation<double>;
    return Center(
      child: Opacity(
        opacity: _opacityTween.evaluate(animation),
        child: Container(
          margin: const EdgeInsets.symmetric(vertical: 10),
          height: _sizeTween.evaluate(animation),
          width: _sizeTween.evaluate(animation),
          child: const FlutterLogo(),
        ),
      ),
    );
  }
}

class LogoApp extends StatefulWidget {
  const LogoApp({super.key});

  @override
  State<LogoApp> createState() => _LogoAppState();
}

class _LogoAppState extends State<LogoApp> with SingleTickerProviderStateMixin {
  late Animation<double> animation;
  late AnimationController controller;

  @override
  void initState() {
    super.initState();
    // #docregion AnimationController, tweens
    controller =
        AnimationController(duration: const Duration(seconds: 2), vsync: this);
    // #enddocregion AnimationController, tweens
    animation = CurvedAnimation(parent: controller, curve: Curves.easeIn)
      ..addStatusListener((status) {
        if (status == AnimationStatus.completed) {
          controller.reverse();
        } else if (status == AnimationStatus.dismissed) {
          controller.forward();
        }
      });
    controller.forward();
  }

  @override
  Widget build(BuildContext context) => AnimatedLogo(animation: animation);

  @override
  void dispose() {
    controller.dispose();
    super.dispose();
  }
}
// #enddocregion diff

// Extra code used only in the tutorial explanations. It is not used by the app.

class UsedInTutorialTextOnly extends _LogoAppState {
  UsedInTutorialTextOnly() {
    // ignore: unused_local_variable, prefer_typing_uninitialized_variables
    var animation, sizeAnimation, opacityAnimation, tween, colorTween;

    // #docregion CurvedAnimation
    animation = CurvedAnimation(parent: controller, curve: Curves.easeIn);
    // #enddocregion CurvedAnimation

    // #docregion tweens
    sizeAnimation = Tween<double>(begin: 0, end: 300).animate(controller);
    opacityAnimation = Tween<double>(begin: 0.1, end: 1).animate(controller);
    // #enddocregion tweens

    // #docregion tween
    tween = Tween<double>(begin: -200, end: 0);
    // #enddocregion tween

    // #docregion colorTween
    colorTween = ColorTween(begin: Colors.transparent, end: Colors.black54);
    // #enddocregion colorTween
  }

  usedInTutorialOnly1() {
    // #docregion IntTween
    AnimationController controller = AnimationController(
        duration: const Duration(milliseconds: 500), vsync: this);
    Animation<int> alpha = IntTween(begin: 0, end: 255).animate(controller);
    // #enddocregion IntTween
    return alpha;
  }

  usedInTutorialOnly2() {
    // #docregion IntTween-curve
    AnimationController controller = AnimationController(
        duration: const Duration(milliseconds: 500), vsync: this);
    final Animation<double> curve =
        CurvedAnimation(parent: controller, curve: Curves.easeOut);
    Animation<int> alpha = IntTween(begin: 0, end: 255).animate(curve);
    // #enddocregion IntTween-curve
    return alpha;
  }
}

// #docregion ShakeCurve
class ShakeCurve extends Curve {
  @override
  double transform(double t) => sin(t * pi * 2);
}
```

---

===================================

## AnimatedBuilder or AnimatedWidget

[source](https://medium.com/flutter/when-should-i-useanimatedbuilder-or-animatedwidget-57ecae0959e8)

**usecase**
To make this more concrete, let’s walk through a specific scenario: I want to write an app with an alien spaceship and have a spaceship beam animation.

`clipper.dart`

```dart
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
        home: Scaffold(
      body: MyHomePage(),
    ));
  }
}

class MyHomePage extends StatelessWidget {
  final Image starsBackground = Image.asset(
    'assets/milky-way.jpg',
  );
  final Image ufo = Image.asset('assets/ufo.png');
  @override
  Widget build(BuildContext context) {
    return Stack(
      alignment: AlignmentDirectional.center,
      children: <Widget>[
        starsBackground,
        ClipPath(
          clipper: const BeamClipper(),
          child: Container(
            height: 1000,
            decoration: BoxDecoration(
              gradient: RadialGradient(
                radius: 1.5,
                colors: [
                  Colors.yellow,
                  Colors.transparent,
                ],
              ),
            ),
          ),
        ),
        ufo,
      ],
    );
  }
}

class BeamClipper extends CustomClipper<Path> {
  const BeamClipper();

  @override
  getClip(Size size) {
    return Path()
      ..lineTo(size.width / 2, size.height / 2)
      ..lineTo(size.width, size.height)
      ..lineTo(0, size.height)
      ..lineTo(size.width / 2, size.height / 2)
      ..close();
  }

  /// Return false always because we always clip the same area.
  @override
  bool shouldReclip(CustomClipper oldClipper) => false;
}
```

**AnimatedBuilder**

To make the beam animate, I am going to wrap this gradient code in an AnimatedBuilder widget. The gradient code goes in the builder function, which gets called when the AnimatedBuilder builds.

Next I need to add a controller to drive this animation. The controller provides the values that AnimatedBuilder uses to draw new versions of the animation frame by frame. As you saw in the previous article, I mix in the SingleTickerProviderStateMixin class and instantiate the controller in the initState function so that it only gets created once. I create the controller in initState, rather than the build method, because I don’t want to create this controller multiple times — I want it to provide new values to animate with for each frame! Because I created a new object in initState, I add a dispose method and tell Flutter that it can get rid of that controller when the parent widget is no longer on the screen.

Then, I pass that controller to the AnimatedBuilder, and my animation runs as expected!

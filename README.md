# Kegelstumpf-Ausmerzung

**veranschaulichen:**

Öffnen Sie index.html nach dem Öffnen dieses Projekts nicht direkt. Sie müssen einen Webserver im Projektverzeichnis ausführen, da sonst eine Ausnahme beim Laden des Modells auftritt. Alternativ können Sie direkt auf https://closecv.com/frustumculling zugreifen. Wenn Sie direkt online auf dieses Projekt zugreifen, bitten wir Sie, aufgrund der Größe des Modells aufgrund von Netzwerkgeschwindigkeitsbeschränkungen geduldig zu warten, bis die Webseite geöffnet wird (dies kann länger als zehn Sekunden dauern ... Das Modell ist 13 m groß).



**Zweck des Experiments:**

1. Implementieren Sie den Frustum-Culling-Algorithmus
2. Probieren Sie verschiedene Auslesemethoden (AABB/OBB...) aus, messen Sie die Leistung und stellen Sie experimentelle Daten bereit

**Versuchsprinzip:**

Die Form des Kegelstumpfs (oder genauer gesagt des Frustums) ähnelt einer Pyramide mit abgeflachter Spitze. Genauer gesagt handelt es sich um eine vierseitige Pyramide mit einer Clipping-Ebene (siehe Abbildung 1), die den unteren Teil ihres Scheitelpunkts abschneidet. Der Kegelstumpf selbst besteht aus sechs Flächen. Diese sechs Flächen werden als nahe Clipping-Ebene, ferne Clipping-Ebene, obere Clipping-Ebene, untere Clipping-Ebene, linke Clipping-Ebene und rechte Clipping-Ebene bezeichnet. Frustum-Clipping ist lediglich ein Prozess, mit dem bestimmt wird, ob ein Objekt gezeichnet werden muss. Obwohl Frustum-Culling dreidimensional sein sollte, lässt es sich in den meisten Fällen mit rein algebraischen Methoden lösen. Und es wird vor der Rendering-Pipeline durchgeführt, im Gegensatz zum Backface-Culling, das nach der Rendering-Pipeline Scheitelpunkt für Scheitelpunkt berechnet werden muss. Die Grafik-Engine sendet die abgeschnittenen Objekte nicht an die Grafikkarte, daher verbessert Frustum-Culling die Rendergeschwindigkeit enorm. Schließlich ist Nichts-Rendering das schnellste Rendering.

![](http://static.oschina.net/uploads/img/201310/20020542_5jKt.png)

**Experimenteller Inhalt:**

Aufgrund begrenzter Energie wurde dieses Experiment mit **three.js** durchgeführt und die verwendeten wichtigen Klassen waren **Frustum** und **Boxhelper**.

Die Frustum-Klasse ist eine Klasse, die den Kegelstumpf und die zugehörigen Methoden kapselt. Beispielsweise können die Methoden intersectsBox und intersectsSphere leicht die Positionsbeziehung zwischen dem Kegelstumpf und dem Begrenzungsrahmen bzw. der Begrenzungskugel ermitteln. Boxhelper kann den Begrenzungsrahmen oder die Begrenzungskugel des Objekts leicht finden. Als Nächstes müssen Objekte außerhalb des Sichtkegels ausgesondert werden.



Zugehöriger Quellcode

Konstruieren Sie Frustum mithilfe der Projektionsmatrix der Kamera und der Inversen der Transformationsmatrix des Weltkoordinatensystems:

```js
setFromMatrix: Funktion ( m ) {

 var planes = diese.planes;
 var me = m.elements;
 var me0 = mich[ 0 ], me1 = mich[ 1 ], me2 = mich[ 2 ], me3 = mich[ 3 ];
 var me4 = me[ 4 ], me5 = me[ 5 ], me6 = me[ 6 ], me7 = me[ 7 ];
 var me8 = me[ 8 ], me9 = me[ 9 ], me10 = me[ 10 ], me11 = me[ 11 ];
 var me12 = me[ 12 ], me13 = me[ 13 ], me14 = me[ 14 ], me15 = me[ 15 ];

 Ebenen[ 0 ].setComponents( me3 - me0, me7 - me4, me11 - me8, me15 - me12 ).normalisieren();
 Ebenen[ 1 ].setComponents( me3 + me0, me7 + me4, me11 + me8, me15 + me12 ).normalisieren();
 Ebenen[ 2 ].setComponents( me3 + me1, me7 + me5, me11 + me9, me15 + me13 ).normalisieren();
 Ebenen[ 3 ].setComponents( me3 - me1, me7 - me5, me11 - me9, me15 - me13 ).normalisieren();
 Ebenen[ 4 ].setComponents( me3 - me2, me7 - me6, me11 - me10, me15 - me14 ).normalize();
 Ebenen[ 5 ].setComponents( me3 + me2, me7 + me6, me11 + me10, me15 + me14 ).normalisieren();

 gib dies zurück;

 }
```



Bestimmen Sie, ob sich der Kegelstumpf und die umgebende Kugel schneiden:

```js
intersectsSphere: Funktion (Kugel) {

 var planes = diese.planes;
 var Zentrum = Kugel.Zentrum;
 var negRadius = - Kugel.Radius;

 für (var i = 0; i < 6; i++) {

 var Abstand = Ebenen[i].AbstandZumPunkt(Mitte);

 wenn (Abstand < negRadius) {

 gibt false zurück;

 }

 }

 gibt true zurück;

 }
```





Bestimmen Sie, ob sich das Kegelstumpf und der Begrenzungsrahmen schneiden:

```js
intersectsBox: Funktion () {

 var p1 = neuer Vektor3(),
 p2 = neuer Vektor3();

 Rückgabefunktion intersectsBox( box ) {

 var planes = diese.planes;

 für (var i = 0; i < 6; i++) {

 var Ebene = Ebenen[i];

 p1.x = Ebene.normal.x > 0 ? Box.min.x : Box.max.x;
 p2.x = Ebene.normal.x > 0 ? Box.max.x : Box.min.x;
 p1.y = Ebene.normal.y > 0 ? Box.min.y : Box.max.y;
 p2.y = Ebene.normal.y > 0 ? Box.max.y : Box.min.y;
 p1.z = Ebene.normal.z > 0 ? Box.min.z : Box.max.z;
 p2.z = Ebene.normal.z > 0 ? Box.max.z : Box.min.z;

 var d1 = plane.distanceToPoint( p1 );
 var d2 = plane.distanceToPoint( p2 );

 // wenn beide außerhalb der Ebene liegen, kein Schnittpunkt

 wenn ( d1 < 0 und d2 < 0 ) {

 gibt false zurück;

 }

 }

 gibt true zurück;

 }
```



**Experimentelle Ergebnisse:**

![](https://ws1.sinaimg.cn/large/006gbcdOgy1frzki4qz13g30z50i3kjl.jpg)

Wie im obigen Bild zu sehen, erscheint beim Öffnen der Weboberfläche ein Scharfschützengewehr am Ursprung des Koordinatensystems. Sie können die Ansicht drehen oder hineinzoomen, um sie anzusehen. Die Bildrate bleibt stabil bei etwa 60. Wenn Sie per Mausklick ein Objekt hinzufügen, werden weitere Scharfschützengewehre angezeigt. Bei über zehn Scharfschützengewehren sinkt die Bildrate deutlich. Aktivieren Sie in diesem Fall den Culling-Algorithmus, um die Bildrate etwas zu verbessern. Wenn die Kamera heranzoomt und nur das Scharfschützengewehr am Ursprungsort beobachtet, kehrt die Bildrate auf 60 Bilder zurück, da alle anderen Scharfschützengewehre eliminiert werden.

**Experimentelle Daten:**

![](https://ws1.sinaimg.cn/large/006gbcdOgy1frzmaihtjfj31lg0dugnh.jpg)

Wir können Folgendes sehen:

1. Wenn Sie nach der Verwendung des Culling-Algorithmus mit der Kamera hineinzoomen, ist die Rendergeschwindigkeit sehr hoch, da die meisten Objekte aussortiert werden.
2. Auch in der Fernansicht wird die Rendergeschwindigkeit deutlich beschleunigt, da einige Objekte ausgeblendet werden können.
3. Im Experiment gibt es keinen offensichtlichen Vorteil oder Nachteil zwischen AABB-Begrenzungsrahmen und Begrenzungskugel.

** Experimentelle Referenz: **

1. [Frustum – three.js-Dokumente]: https://threejs.org/docs/#api/math/Frustum

2. [THREE.js prüft, ob sich ein Objekt im Kegelstumpf befindet – Stack Overflow]: https://stackoverflow.com/questions/24877880/three-js-check-if-object-is-in-frustum

3. [Wie bestimmt three.js, ob eine Koordinate im Sichtfeld der Kamera liegt? - Zhihu]: https://www.zhihu.com/question/49989787

4. [[3D-Grafik\] Einführung in Frustum Culling (Übersetzung) - szszss]: https://my.oschina.net/u/999400/blog/170062

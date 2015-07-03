---
ID: 436
post_title: 'Dématérialisation &#8211; Faire signer sur support tactile'
author: Guillaume Gerbaud
post_date: 2011-09-09 09:53:00
post_excerpt: "<p>Avec l’avènement des appareils tactiles, les solutions de dématérialisation se multiplient. Certains documents sont plus difficilement dématérialisables, c'est le cas en particulier des documents nécessitant une signature. Il existe évidemment des solutions de signature électronique mais elles ne peuvent pas être mise en place dans toutes les situations.<br /></p> <p>Imaginons le cas d'un technicien se déplaçant au domicile de son client pour réaliser une intervention, faire un rapport de cette dernière et enfin faire signer ce dernier au client. Il peut s'avérer intéressant, aujourd'hui, de saisir ce rapport sur tablette tactile.<br /></p> <p>Dans ce billet, je vais vous présenter comment créer un composant de signature pour appareils tactiles Android.</p>"
layout: post
permalink: http://blog.zenika-offres.com/?p=436
published: true
slide_template:
  - ""
---
<p>Avec l’avènement des appareils tactiles, les solutions de dématérialisation se multiplient. Certains documents sont plus difficilement dématérialisables, c'est le cas en particulier des documents nécessitant une signature. Il existe évidemment des solutions de signature électronique mais elles ne peuvent pas être mise en place dans toutes les situations.<br /></p> <p>Imaginons le cas d'un technicien se déplaçant au domicile de son client pour réaliser une intervention, faire un rapport de cette dernière et enfin faire signer ce dernier au client. Il peut s'avérer intéressant, aujourd'hui, de saisir ce rapport sur tablette tactile.<br /></p> <p>Dans ce billet, je vais vous présenter comment créer un composant de signature pour appareils tactiles Android.</p>
<!--more-->
<p><img src="/wp-content/uploads/2015/07/.device-2011-09-01-220506_m.jpg" alt="ZenSignature - Home" style="display:block; margin:0 auto;" title="ZenSignature - Home" /></p> <h2>View, le composant de base</h2> <p><br />
Lorsqu'on veut créer un nouveau composant visuel, celui-ci doit hériter de la classe View ou d'une de ses classes fille. <br /></p> <pre> public class SignatureView extends View {     public SignatureView(Context c, AttributeSet set)     {         super(c,set);      } } </pre> <p><br />
Une chose importante à savoir est que lorsque le constructeur de votre composant est appelé, sa taille initiale est de 0 sur 0. Vous ne pouvez donc pas y faire les initialisations nécessitant de connaître la taille réelle de votre composant. <br />
Par contre, la méthode onSizeChanged vous informe dès que votre composant change de taille et notamment, lorsque celui-ci est inséré dans la hiérarchie des vues de l'activité. C'est donc ici que nous allons initialiser nos éléments. <br />
On veut pouvoir dessiner sur notre composant mais aussi pouvoir sauvegarder le dessin. Nous allons donc garder une référence sur le Bitmap du Canvas. <br /></p> <pre>     private static final int BACKGROUND = Color.WHITE;     private Bitmap mBitmap;     private Canvas mCanvas;     @Override     protected void onSizeChanged(int w, int h, int oldw, int oldh)     {     	mBitmap = Bitmap.createBitmap(w, h, Bitmap.Config.ARGB_8888);         mCanvas = new Canvas();         mCanvas.setBitmap(mBitmap);         Paint p = new Paint();         p.setColor(BACKGROUND);         mCanvas.drawPaint(p);     } </pre> <h2>Saisir la signature</h2> <p><br />
Nous allons utiliser la méthode onTouchEvent de la classe View. Cette méthode est appelée dès que l'utilisateur touche l'écran. L'unique paramètre de cette méthode est un objet MotionEvent. La méthode MotionEvent#getAction nous indique le type de mouvement détecté. Les 2 actions qui nous intéresse sont ACTION_DOWN (un appui) et ACTION_MOVE (un déplacement). Nous allons donc suivre tous les mouvements de l'utilisateur afin de dessiner son tracé. <br />
Dans un souci évident de performances, le système n'appelle pas la méthode onTouchEvent à chaque position de l'utilisateur, mais "bufferise" celles-ci. Ainsi, l'objet MotionEvent contient les coordonnées du point courant où se situe l'utilisateur mais également un historique des points précédents depuis le précédent appel de onTouchEvent.<br />
Voici donc à quoi ressemble notre méthode onTouchEvent. <br /></p> <pre>     @Override     public boolean onTouchEvent(MotionEvent event)     {         int action = event.getAction();         if(action == MotionEvent.ACTION_MOVE || action == MotionEvent.ACTION_DOWN)         {         	int n = event.getHistorySize();         	for (int i=0; i&lt;n; i++)         	{         		// action here with event.getHistoricalX(i) and event.getHistoricalY(i)         	}         	// action here with event.getX() and event.getY()         }         return true;     } </pre> <p><br />
Nous allons à présent dessiner ce tracé. Lorsque l'action est ACTION_DOWN, nous dessinons un point aux coordonnées de l’évènement, par contre, lors d'un déplacement, nous devons relier ces points afin de dessiner tout le trajet. Pour cela, nous avons besoin de connaître en permanence le point précédent afin de le relier au point courant. <br /></p> <pre>     private static final int SIZE = 4;     private static final int FOREGROUND = Color.BLACK;     private float mCurX;     private float mCurY;     @Override     public boolean onTouchEvent(MotionEvent event)     {         int action = event.getAction();         boolean line = action == MotionEvent.ACTION_MOVE;         if(line || action == MotionEvent.ACTION_DOWN)         {         	final Paint p = new Paint();         	p.setColor(FOREGROUND);             p.setAntiAlias(true);         	p.setStrokeWidth(SIZE);         	int n = event.getHistorySize();         	for (int i=0; i&lt;n; i++)         	{         		drawPoint(event.getHistoricalX(i), event.getHistoricalY(i), line, p);         	}         	drawPoint(event.getX(), event.getY(), line, p);         }         return true;     }     private void drawPoint(float x, float y, boolean line, Paint p)     {         if (mBitmap != null)         {             if(line)             {             	mCanvas.drawLine(mCurX, mCurY, x, y, p);             }             else             {             	mCanvas.drawCircle(x, y, SIZE/2, p);             }             invalidate();         }         mCurX = x;         mCurY = y;     } </pre> <h2>Effacer l'écran, récupérer l'image</h2> <p><br />
Nous ajoutons 2 méthodes publiques à notre composant afin de pouvoir effacer l'écran et récupérer l'image. Pour effacer, on repeint tout simplement avec la couleur de fond. <br /></p> <pre>     public void clear()     {         if (mCanvas != null)         {             reset();             invalidate();         }     }     @Override     protected void onSizeChanged(int w, int h, int oldw, int oldh)     {     	mBitmap = Bitmap.createBitmap(w, h, Bitmap.Config.ARGB_8888);         mCanvas = new Canvas();         mCanvas.setBitmap(mBitmap);     	reset();     }     private void reset()     {         Paint p = new Paint();         p.setColor(BACKGROUND);         mCanvas.drawPaint(p);     }     public Bitmap save()     {     	return mBitmap;     } </pre> <p><img src="/wp-content/uploads/2015/07/.device-2011-09-02-152002_noheader_s.jpg" alt="ZenSignature - Signature" style="display:block; margin:0 auto;" title="ZenSignature - Signature" /></p> <h2>Sauvegarder en base de données</h2> <p><br />
Il est ensuite possible de sauvegarder cette image dans une base de données SQLite, dans une colonne de type BLOB. <br /></p> <pre>         // get Bitmap from SignatureView     	Bitmap b = mSignatureView.save();     	// calculate the number of pixels     	int pixels = b.getHeight() * b.getRowBytes();     	// instanciate an OutputStream to handle the pixels     	ByteArrayOutputStream baos = new ByteArrayOutputStream(pixels);     	// we record it as a PNG (lossless format)     	b.compress(CompressFormat.PNG, 0, baos);     	// we get the bytes array     	byte[] bytes = baos.toByteArray();     	// ContentValues is a wrapper class used to insert in db     	ContentValues cv = new ContentValues(1);     	cv.put(Signature.SIGNATURE, bytes);     	// here we insert in database, and we get the uri of the image     	Uri uri = mContext.getContentResolver().insert(Signature.CONTENT_URI, cv); </pre> <h2>Restaurer l'image</h2> <p><br />
Enfin, voyons comment afficher l'image après l'avoir récupérée depuis la base de données <br /></p> <pre>         // request with the image uri, so no selection clause needed         Cursor result = mContext.getContentResolver().query( 					uri, 					Signature.PROJECTION, 					null, 					null, 					Signature.DEFAULT_SORT_ORDER);         // we check we got something 	if(result != null &amp;&amp; result.moveToFirst()) 	{                 // we retrieve the raw picture 		byte[] rawPict = result.getBlob(Signature.SIGNATURE_INDEX);                 // then get a Bitmap from it 		Bitmap b = BitmapFactory.decodeByteArray(rawPict, 0, rawPict.length);                 // just set the Bitmap in an ImageView 		mImageView.setImageBitmap(b); 	} </pre> <h2>Code Source</h2> <p><br />
Les sources de l'application sont disponibles sur le <a href="https://github.com/Zenika/Blogs/tree/master/20110909_Dematerialisation-Faire-signer-sur-support-tactile">Github de Zenika</a><br /></p>
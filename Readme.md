# Un exemple minimaliste d'une application JavaEE 7

Pour faire simple l'application consiste à manipuler des comptes bancaires (très simplifiés).

Un compte est identifié par un numéro (alphanumérique) et le montant du compte.

On peut :
- créer des comptes,
- créditer un compte.

Les comptes sont stockés dans une base de données. Une entité (JPA) permet d'y accéder (cf. partie EJBs).

Un EJB session sans état (stateless) sert de façade pour les opérations bancaires.

Un client WEB permet de réaliser l'ensemble des opérations sur les comptes : création, crédit ou visualisation (cf. client Web).
Ce client comporte une page jsp et trois servlet. La connexion au bean session se fait à l'aide de l'interfaces distante de l'EJB.

## Partie EJB

L'object persistant, Compte, ainsi que l'EJB session et son interface sont regroupés ensembles dans la même archive.
<pre>
CompteEjb.jar (ejb-jar)
  |-- <a href="CompteEjb/src/main/java/fr/usmb/m2isc/javaee/comptes/jpa/Compte.java" >fr/usmb/m2isc/.../jpa/Compte.class</a> (implantation de l'entité Compte (entité JPA))
  |-- <a href="CompteEjb/src/main/java/fr/usmb/m2isc/javaee/comptes/ejb/OperationBean.java" >fr/usmb/m2isc/.../ejb/OperationBean.class</a> (implantation de l'enterprise java bean Operation (bean session))
  |-- <a href="CompteEjb/src/main/java/fr/usmb/m2isc/javaee/comptes/ejb/Operation.java" >fr/usmb/m2isc/.../ejb/Operation.class</a> (interfaces de manipulation distante du bean session)
  |-- META-INF/MANIFEST.MF (java manifeste)
  |-- META-INF/ejb-jar.xml (descripteur standard des enterprise java beans -- optionnnel dans les dernières versions de javaEE)
  |-- <a href="CompteEjb/src/main/resources/META-INF/persistence.xml" >META-INF/persistence.xml</a> (descripteur standard pour JPA)
  |-- META-INF/orm.xml (descripteur pour le mapping objet-relationel  -- absent ici)
</pre>

Toutes les manipulations sur les objets persistants se font dans l'EJB en utilisant l'_entity manager_ correspondant à l'_unité de persistance_ des objets persistants manipulés. 

Dans l'EJB on utilise l'annotation `@PersistenceContext` pour récupérer auprès du serveur JavaEE l'_entity manager_ désiré.

```java
@Stateless
@Remote
public class OperationBean implements Operation {
	
	@PersistenceContext
	private EntityManager em;
```

## L'application WEB est dans un fichier d'archive war :

Ce client WEB permet de réaliser l'ensemble des opérations sur les comptes : création, crédit ou visualisation.

Ce client comporte des pages jsp et des servlet. La connexion au bean session se fait à l'aide de l'interface distante de l'EJB. 
On utilise les _servlet_ pour traiter les requêtes et les _pages JSP_ pour l'affichage du résultat.
<pre>
CompteWeb.war
  |-- <a href="CompteWeb/src/main/webapp/index.html" >index.html</a> (page d'accueil -- formulaires html permettant de créer, visualiser ou modifier les comptes)
  |-- <a href="CompteWeb/src/main/webapp/AfficherCompte.jsp" >AfficherCompte.jsp</a> (page jsp pour afficher un compte)
  |-- <a href="CompteWeb/src/main/webapp/META-INF/MANIFEST.MF" >META-INF/MANIFEST.MF</a> (java manifeste)
  |-- WEB-INF/classes (classes java pour les servlets :
                |-- <a href="CompteWeb/src/main/java/fr/usmb/m2isc/javaee/comptes/web/CreerCompteServlet.java" >fr/usmb/m2isc/javaee/comptes/web/CreerCompteServlet.class</a>
                |-- <a href="CompteWeb/src/main/java/fr/usmb/m2isc/javaee/comptes/web/CrediterCompteServlet.java" >fr/usmb/m2isc/javaee/comptes/web/CrediterCompteServlet.class</a>
                |-- <a href="CompteWeb/src/main/java/fr/usmb/m2isc/javaee/comptes/web/AfficherCompteServlet.java" >fr/usmb/m2isc/javaee/comptes/web/AfficherCompteServlet.java.class</a>
  |-- WEB-INF/lib (librairies java utilisées dans les servlet)
  |-- <a href="CompteWeb/src/main/webapp/WEB-INF/web.xml" >WEB-INF/web.xml</a> (descripteur standard de l'application Web -- optionnnel dans les dernières versions de javaEE)
</pre>

Dans les _servlet_ on utilise l'annotation `@EJB` pour obtenir une référence de l'_EJB session_ :

```java
@WebServlet("/CreerCompteServlet")
public class CreerCompteServlet extends HttpServlet {
	private static final long serialVersionUID = 1L;
	
	@EJB
	private Operation ejb;
```

L'EJB est ensuite utilisé par les servlet pour effectuer les traitements :

```java
	protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		// recuperation des parametres de la requete
		String num = request.getParameter("numero");
		String val = request.getParameter("depot");
		double depot = Double.parseDouble(val);
		
		// utilisation de l'EJB
		Compte cpt = ejb.creerCompte(num, depot);
		...
```

puis on utilise la requête pour passer les objets à afficher à la _page JSP_ chargée de l'affichage :

```java
		...
		// ajout du compte dans la requete
		request.setAttribute("compte", cpt);
		// redirection vers la page JSP pour afficher le compte
		request.getRequestDispatcher("/AfficherCompte.jsp").forward(request, response);		
	}
```

## Le tout est packagé ensemble dans une archive ear :

Cette archive permet de regouper dans le même fichier l'ensemble des composants de l'application (ejb, app web, etc.).
<pre>
CompteEar.ear
  |-- CompteEjb.jar (archive contenant les EJBs)
  |-- CompteWeb.war (archive contenant le client Web)
  |-- <a href="CompteEar/src/main/resources/META-INF/application.xml" >META-INF/application.xml</a> (descripteur standard de l'application -- optionnnel dans les dernières versions de javaEE)
</pre>

## Usage :

Pour voir les sources il suffit de cloner le projet git et de l'importer (sous forme de projet gradle) dans votre IDE favori. 
Cela devrait permettre la création de 3 sous-projets (ou modules), un pour la partie EJB et JPA , un pour la partie WEB et un pour la partie EAR.

La création des archives (CompteWeb.war, CompteEjb.jar, CompteEar.ear) peut se faire via gradle en appelant la tâche build sur le projet principal.

Pour utiliser l'exemple il suffit de déployer le fichier CompteEar.ear sur un serveur JavaEE 7. 
Le client Web est alors dans déployé dans _/CompteWeb_. 
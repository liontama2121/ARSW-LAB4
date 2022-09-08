### Escuela Colombiana de Ingeniería

### Arquitecturas de Software
## Laboratorio Componentes y conectores  Middleware- gestión de planos
### Dependencias
* [Ejercicio introductorio al manejo de Spring y la configuración basada en anotaciones](https://github.com/ARSW-ECI-beta/DIP_DI-SPRING_JAVA-GRAMMAR_CHECKER).

### Descripción
En este ejercicio se va a construír un modelo de clases para la capa lógica de una aplicación que permita gestionar planos arquitectónicos de una prestigiosa compañia de diseño. 

![](img/ClassDiagram1.png)

### Parte I.

1. Configure la aplicación para que funcione bajo un esquema de inyección de dependencias, tal como se muestra en el diagrama anterior.


	Lo anterior requiere:

	* Agregar las dependencias de Spring.
	
	``` xml
	    <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>5.3.9</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.3.9</version>
        </dependency>
	```
	
	* Agregar la configuración de Spring.
	
	``` xml
	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
       		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       		xmlns:context="http://www.springframework.org/schema/context"
       		xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
        		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.2.xsd">
    			<context:component-scan base-package="edu.eci.arsw" />	
	</beans>
	```
	
	* Configurar la aplicación -mediante anotaciones- para que el esquema de persistencia sea inyectado al momento de ser creado el bean 'BlueprintServices'.
	
		
	> Se configuran la anotación de la clase ```InMemoryBlueprintPersistence``` como servicio
		
		```java
		@Service("InMemoryBlueprintPersistence")
		public class InMemoryBlueprintPersistence implements BlueprintsPersistence{
		```
		
	> Se configuran la anotación de la clase ```BlueprintsServices``` como servicio y se invocan los servicios de ```InMemoryBlueprintPersistence``` y ```RedundancyFilter```
		
		```java
		@Service
		public class BlueprintsServices {
   
    			@Autowired
    			@Qualifier("InMemoryBlueprintPersistence")
    			BlueprintsPersistence bpp=null;

    			@Autowired
    			@Qualifier("RedundancyFilter")
    			BlueprintsFilter bpf;
		```
	


2. Complete los operaciones getBluePrint() y getBlueprintsByAuthor(). Implemente todo lo requerido de las capas inferiores (por ahora, el esquema de persistencia disponible 'InMemoryBlueprintPersistence') agregando las pruebas correspondientes en 'InMemoryPersistenceTest'.

	Se implementa el metodo ```getBluePrint()``` en la clase ```InMemoryBlueprintPersistence```
	
	```java
	@Override
    	public Blueprint getBlueprint(String author, String bprintname) throws BlueprintNotFoundException {
        	return blueprints.get(new Tuple<>(author, bprintname));
    	}
	```
	
	Se implementa el metodo ```getBlueprintsByAuthor()``` en la clase ```InMemoryBlueprintPersistence```
	
	```java
	@Override
    	public Set<Blueprint> getBlueprintsByAuthor(String author) throws BlueprintNotFoundException {
        	Set<Blueprint> ans = new HashSet<>();
        	Set<Tuple<String,String>> llaves = blueprints.keySet();
        	for(Tuple<String,String> i : llaves){
            		if(i.getElem1().equalsIgnoreCase(author)){
		                ans.add(blueprints.get(i));
            		}
        	}
        	return ans;
    	}
	```
	
	Después en la clase ```BlueprintsServices``` completamos el metodo ```getAllBlueprints()```
	
	```java
	public Set<Blueprint>getAllBlueprints();
	```
	
	Después en la clase ```BlueprintsServices``` completamos el metodo ```getBlueprintsByAuthor()```
	
	```java
	public Set<Blueprint> getBlueprintsByAuthor(String author) throws BlueprintNotFoundException;
	```
	
	Luego implementamos la prueba en ``````
	
	```java
	@Test
    	public void getBlueprintsByAuthorTest(){
	        InMemoryBlueprintPersistence ibpp=new InMemoryBlueprintPersistence();
        	Point[] pts0=new Point[]{new Point(40, 40),new Point(15, 15)};
	        Blueprint bp0=new Blueprint("mack", "mypaint",pts0);
        	Point[] pts1=new Point[]{new Point(0, 0),new Point(10, 10)};
        	Blueprint bp1=new Blueprint("john", "thepaint",pts1);
        	Point[] pts2=new Point[]{new Point(0, 0),new Point(10, 10)};
        	Blueprint bp2=new Blueprint("mack", "thepaint1",pts2);
        	Point[] pts3=new Point[]{new Point(0, 0),new Point(10, 10)};
        	Blueprint bp3=new Blueprint("john", "thepaint1",pts3);
        	Set<Blueprint> res= null;
        	try {
            		ibpp.saveBlueprint(bp0);
            		ibpp.saveBlueprint(bp1);
            		ibpp.saveBlueprint(bp2);
            		ibpp.saveBlueprint(bp3);
        	} catch (BlueprintPersistenceException e) {
            		e.printStackTrace();
        	}
        	Set<Blueprint> aux = new HashSet<>();
        	aux.add(bp0);
        	aux.add(bp2);
        	try {
            		res = ibpp.getBlueprintsByAuthor("mack");
        	} catch (BlueprintNotFoundException e) {
            		e.printStackTrace();
        	}
        	assertEquals(aux,res);
    	}
	```
	

3. Haga un programa en el que cree (mediante Spring) una instancia de BlueprintServices, y rectifique la funcionalidad del mismo: registrar planos, consultar planos, registrar planos específicos, etc.

	Para rectificar la funiconalidad de registrar planos se creó el método ```addNewBlueprint()``` en la clase ```BlueprintsServices```
	
	```java
	public void addNewBlueprint(Blueprint bp) throws BlueprintPersistenceException {
     	   bpp.saveBlueprint(bp);
    	}
	```
	
	Para rectificar la funcionalidad de consultar planos, se creó el método ```getAllBlueprints()``` en la clase ```BlueprintsServices```
	
	```java
	public Set<Blueprint> getAllBlueprints(){
       		return bpp.getAllBlueprints();
   	}
	```
	
	Despues realizamos la modificación del metodo ```getBlueprint()```
	
	```java
	public Blueprint getBlueprint(String author,String name) throws BlueprintNotFoundException{
 	       return bpp.getBlueprint(author,name);
	}
	```
	
	y también modificamos el metodo ```getBlueprintsByAuthor()```
	
	```java
	public Set<Blueprint> getBlueprintsByAuthor(String author) throws BlueprintNotFoundException{
 	       return bpp.getBlueprintsByAuthor(author);
	}
	```
	

	

4. Se quiere que las operaciones de consulta de planos realicen un proceso de filtrado, antes de retornar los planos consultados. Dichos filtros lo que buscan es reducir el tamaño de los planos, removiendo datos redundantes o simplemente submuestrando, antes de retornarlos. Ajuste la aplicación (agregando las abstracciones e implementaciones que considere) para que a la clase BlueprintServices se le inyecte uno de dos posibles 'filtros' (o eventuales futuros filtros). No se contempla el uso de más de uno a la vez:
	* (A) Filtrado de redundancias: suprime del plano los puntos consecutivos que sean repetidos.
	* (B) Filtrado de submuestreo: suprime 1 de cada 2 puntos del plano, de manera intercalada.

	Se realiza la creación de la interfaz ```BlueprintsFilter``` para que la consulta de planos se realicen con un proceso de filtrado, antes de retornar los planos consultados, para así reducir el tamaño de los planos, removiendo datos redundantes o simplemente submuestrando, antes de retornarlos

		```java
		public interface BlueprintsFilter {
 			public Blueprint filter(Blueprint bp);
		}
		```
	
	Para realizar el filtrado por redundancias, que se encargara de suprimir del plano los puntos consecutivos que sean repetidos, se crea la clase ```RedundancyFilter```, que se encarga de realizar dichas operaciones; la cual extiende de ```BlueprintsFilter```

		```java
		@Service("RedundancyFilter")
		public class RedundancyFilter implements BlueprintsFilter {
    			@Override
			public Blueprint filter(Blueprint bp) {
        			ArrayList<Point> points=new ArrayList<Point>();
        			for (Point i :bp.getPoints()){
            				boolean found=false;
            				for(Point j : points){
                				if(i.equals(j)){
                    					found=true;
                    					break;
                				}
            				}
            				if(!found)points.add(i);
        			}
        			return new Blueprint(bp.getAuthor(),bp.getName(),points);
  			}
		}
		```
		
	
	Para realizar el filtrado de submuestreo, para poder suprimir 1 de cada 2 puntos del plano, de manera intercalada, se crea la clase ```SubsamplingFilter```, que se encarga de realizar dichas operaciones; la cual extiende de ```BlueprintsFilter```
	
		```java
		@Service("SubsamplingFilter")
		public class SubsamplingFilter implements BlueprintsFilter {
    			@Override
	    		public Blueprint filter(Blueprint bp) {
        			List<Point> oldPoints=bp.getPoints();
        			ArrayList<Point> points=new ArrayList<Point>();
        			for(int i=0;i<oldPoints.size();i++){
		            		if(i%2==0){
                				points.add(oldPoints.get(i));
            				}
        			}
        			return new Blueprint(bp.getAuthor(),bp.getName(),points);
    			}

		}
		```
	
	En la clase ```Blueprint``` modificamos el constructor ```Blueprint(String author,String name,List<Point> pnts)``` y lo dejamos de la siguiente manera
	
		```java
		public Blueprint(String author,String name,List<Point> pnts){
       			this.author=author;
        		this.name=name;
       			points=pnts;
		}
		```
		
	Y en la misma clase ```Blueprint``` modificamos el constructor ```Blueprint(String author, String name)``` y lo dejamos de la siguiente manera
	
		```java
		public Blueprint(String author, String name){
       			this.name=name;
       			this.author=author;
        		points=new ArrayList<>();
		}
		```
	
	Por último, se crea un metodo ```main``` en la clase ```Main``` encargado de la realizar la ejecución del programa
	
		```java
		public class Main {
        public static void main(String[] args) throws BlueprintPersistenceException, BlueprintNotFoundException {
        ApplicationContext ac = new ClassPathXmlApplicationContext("applicationContext.xml");
        BlueprintsServices bps = ac.getBean(BlueprintsServices.class);
        bps.addNewBlueprint(new Blueprint("GOYA","bp1"));
        Blueprint bp1=bps.getBlueprint("GOYA","bp1");
        System.out.println(bp1);
        bps.addNewBlueprint(new Blueprint("hitler","bp2"));
        System.out.println(bps.getBlueprintsByAuthor("hitler"));
        Point[] points= new Point[] {new Point(1,2),new Point(3,4),new Point(1,2)};
        bps.addNewBlueprint(new Blueprint("AnaFrank","bp3",points));
        System.out.println(bps.getAllBlueprints());
        Blueprint bp3=bps.getBlueprint("AnaFrank","bp3");
        System.out.println(bps.filter(bp3).getPoints());
    }
}
		```

5. Agrege las pruebas correspondientes a cada uno de estos filtros, y pruebe su funcionamiento en el programa de prueba, comprobando que sólo cambiando la posición de las anotaciones -sin cambiar nada más-, el programa retorne los planos filtrados de la manera (A) o de la manera (B). 

	Creamos el paquete ```filters``` en el interior del paquete ```edu.eci.arsw.blueprints.test```
	
	Al interior de este paquete creamos la clase de pruebas de ```RedundancyFilterTest``` y el metodo de prueba ```filterRepeated()```
	
		```java
		public class RedundancyFilterTest {
    			@Test
    			public void filterRepeated() {
        			RedundancyFilter prueba = new RedundancyFilter();
        			Point puntos[] = {new Point(1, 1), new Point(1, 1), new Point(2, 2), new Point(2, 2), new Point(3, 3), new Point(3, 3)};
        			Blueprint blueprint = new Blueprint("mack", "mypaint", puntos);
        			blueprint = prueba.filter(blueprint);
        			List<Point> resPuntos = new ArrayList<>();
        			resPuntos.add(new Point(1, 1));
        			resPuntos.add(new Point(2, 2));
        			resPuntos.add(new Point(3, 3));
        			assertTrue(blueprint.getPoints().size() == resPuntos.size());
        			List<Point> res = blueprint.getPoints();
        			for (int i = 0; i < res.size(); i++){
            				assertEquals(resPuntos.get(i),res.get(i));
        			}
    			}
		}

		```
	
	Y en el mismo lugar creamos la clase de pruebas ```SubsamplingFilterTest``` y el metodo de prueba ```subsamplingFiltering```
	
		```java
		public class SubsamplingFilterTest {
    			@Test
    			public void subsamplingFiltering(){
        			SubsamplingFilter prueba = new SubsamplingFilter();
        			Point puntos[] = {new Point(1,2), new Point(3,4), new Point(5,6), new Point(7,8), new Point(9,10)};
        			Blueprint blueprint = new Blueprint("mack","mypaint",puntos);
        			blueprint = prueba.filter(blueprint);
        			List<Point> resPuntos = new ArrayList<>();
        			resPuntos.add(new Point(1,2));
        			resPuntos.add(new Point(5,6));
        			resPuntos.add(new Point(9,10));
        			assertTrue(blueprint.getPoints().size() == resPuntos.size());
        			List<Point> res = blueprint.getPoints();
        			for (int i = 0; i < res.size(); i++){
            				assertEquals(resPuntos.get(i),res.get(i));
        			}
    			}
		}
		```
	
### Autores
* [Juan Camilo Molina](https://github.com/liontama2121)

### Licencia & Derechos de Autor
**©️** Juan Camilo Molina Leon, Estudiante de Ingeniería de Sistemas de la Escuela Colombiana de Ingeniería Julio Garavito

Licencia bajo la [GNU General Public License](/LICENSE.txt)
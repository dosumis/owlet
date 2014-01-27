owlet
==========

**owlet** is a query expansion preprocessor for SPARQL. It parses embedded OWL class expressions and uses an OWL reasoner to replace them with FILTER statements containing the URIs of subclasses of the given class expression (or superclasses, equivalent classes, or instances).

**owlet** is written in Scala but can be used in any Java application.

## Installation
owlet is not yet available from an online Maven repository. To use it in your project you will need to:

1. Check out the code, e.g. run `git clone https://github.com/phenoscape/owlet.git` on the command line.
2. Run `mvn install` to build the jar and add to your local Maven repository.
3. Add the owlet dependency to your `pom.xml`: 

```xml
<dependency>
  <groupId>org.phenoscape</groupId>
  <artifactId>owlet</artifactId>
  <version>1.0-SNAPSHOT</version>
</dependency>
```

## Usage
Below is an example of loading an ontology from a file into an OWL reasoner, and then using the reasoner to expand a SPARQL query to a triple store. In this example, the triple store is in-memory and holds the same ontology as the reasoner, but you could replace that part with querying a remote SPARQL endpoint that, for example, contains data linked to the ontology.

```scala
import org.semanticweb.elk.owlapi.ElkReasonerFactory
import org.semanticweb.owlapi.apibinding.OWLManager
import org.semanticweb.owlapi.reasoner.InferenceType
import com.hp.hpl.jena.query.QueryExecutionFactory
import com.hp.hpl.jena.query.QueryFactory
import com.hp.hpl.jena.rdf.model.ModelFactory

val manager = OWLManager.createOWLOntologyManager()
val ontology = manager.loadOntologyFromOntologyDocument(new File("my_ontology.owl"))
reasoner = new ElkReasonerFactory().createReasoner(ontology)
reasoner.precomputeInferences(InferenceType.CLASS_HIERARCHY)
val rdfModel = ModelFactory.createDefaultModel() 
rdfModel.read("my_ontology.owl")
val queryText = """
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX ow: <http://purl.org/phenoscape/owlet/syntax#>
PREFIX axial_skeleton: <http://purl.obolibrary.org/obo/VSAO_0000056>
PREFIX definition: <http://purl.obolibrary.org/obo/IAO_0000115>
PREFIX part_of: <http://purl.obolibrary.org/obo/BFO_0000050>
SELECT DISTINCT ?structure ?label ?definition
WHERE
{
  ?structure rdfs:label ?label .
  ?structure definition: ?definition .
  ?structure rdfs:subClassOf "part_of: some axial_skeleton:"^^ow:omn .
}
  """
val expander = new QueryExpander(reasoner)
val query = QueryFactory.create(queryText)
val expandedQuery = expander.expandQuery(query)
val results = QueryExecutionFactory.create(expandedQuery, vsaoRDF).execSelect()
```

More documentation can be found on the [owlet wiki](https://github.com/phenoscape/owlet/wiki).

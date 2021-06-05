---
layout: documentation
title: 15 Minutes Tutorial - Extended
part: Getting Started
---

# {{page.title}} {#domainmodel-next-steps}

After you have developed your first own DSL, the question arises how the behavior and the semantics of the language can be customized. Here you find a few mini-tutorials that illustrate common use cases when crafting your own DSL. These lessons are independent from each other. Each of them will be based on the language that was built in the previous [domainmodel tutorial](102_domainmodelwalkthrough.html).

## Writing a Code Generator With Xtend {#tutorial-code-generation}

As soon as you generate the Xtext artifacts from the grammar, a code generator stub is put into the runtime project of your language. Let's dive into [Xtend](https://www.eclipse.org/xtend/) and see how you can integrate your own code generator with Eclipse.

In this lesson you will generate Java Beans for entities that are defined in the domainmodel DSL. For each *Entity*, a Java class is generated and each *Feature* will lead to a private field in that class including public getters and setters. For the sake of simplicity, we will use fully qualified names all over the generated code.

```java
package my.company.blog;

public class HasAuthor {
    private java.lang.String author;

    public java.lang.String getAuthor() {
        return author;
    }

    public void setAuthor(java.lang.String author) {
        this.author = author;
    }
}
```

First of all, locate the file *DomainmodelGenerator.xtend* in the package *org.example.domainmodel.generator*. This Xtend class is used to generate code for your models in the standalone scenario and in the interactive Eclipse environment. Let's make the implementation more meaningful and start writing the code generator. The strategy is to find all entities within a resource and trigger code generation for each one.

1.  First of all, you will have to filter the contents of the resource down to the defined entities. Therefore we need to iterate a resource with all its deeply nested elements. This can be achieved with the method `getAllContents()`. To use the resulting [TreeIterator]({{site.src.emf}}/plugins/org.eclipse.emf.common/src/org/eclipse/emf/common/util/TreeIterator.java) in a `for` loop, we use the extension method `toIterable()` from the built-in library class [IteratorExtensions]({{site.src.xtext_lib}}/org.eclipse.xtext.xbase.lib/src/org/eclipse/xtext/xbase/lib/IteratorExtensions.java).

    ```xtend
    class DomainmodelGenerator extends AbstractGenerator {

        override void doGenerate(Resource resource, IFileSystemAccess2 fsa, IGeneratorContext context) {
            for (e : resource.allContents.toIterable.filter(Entity)) {
                
            }
        }
    }
    ```

1.  Now let's answer the question how we determine the file name of the Java class that each *Entity* should yield. This information should be derived from the qualified name of the *Entity* since Java enforces this pattern. The qualified name itself has to be obtained from a special service that is available for each language. Fortunately, Xtend allows to reuse that one easily. We simply inject the [IQualifiedNameProvider]({{site.src.xtext_core}}/org.eclipse.xtext/src/org/eclipse/xtext/naming/IQualifiedNameProvider.java) into the generator.

    ```xtend
      @Inject extension IQualifiedNameProvider
    ```

    This allows to ask for the name of an entity. It is straightforward to convert the name into a file name:

    ```xtend
    override void doGenerate(Resource resource, IFileSystemAccess2 fsa, IGeneratorContext context) {
        for (e : resource.allContents.toIterable.filter(Entity)) {
            fsa.generateFile(
                e.fullyQualifiedName.toString("/") + ".java",
                e.compile)
        }
    }
    ```

1.  The next step is to write the actual template code for an entity. For now, the function `Entity.compile` does not exist, but it is easy to create it:

    ```xtend
    private def compile(Entity e) '''
        package «e.eContainer.fullyQualifiedName»;
        
        public class «e.name» {
        }
    '''
    ```

1.  This small template is basically the first shot at a Java-Beans generator. However, it is currently rather incomplete and will fail if the *Entity* is not contained in a package. A small modification fixes this. The `package` declaration has to be wrapped in an `IF` expression:

    ```xtend
    private def compile(Entity e) '''
        «IF e.eContainer.fullyQualifiedName !== null»
            package «e.eContainer.fullyQualifiedName»;
        «ENDIF»
        
        public class «e.name» {
        }
    '''
    ```

    Let's handle the *superType* of an *Entity* gracefully, too, by using another `IF` expression:

    ```xtend
    private def compile(Entity e) '''
        «IF e.eContainer.fullyQualifiedName !== null»
            package «e.eContainer.fullyQualifiedName»;
        «ENDIF»
        
        public class «e.name» «IF e.superType !== null
                »extends «e.superType.fullyQualifiedName» «ENDIF»{
        }
    '''
    ```

1.  Even though the template will compile the *Entities* without any complaints, it still lacks support for the Java properties that each of the declared features should yield. For that purpose, you have to create another Xtend function that compiles a single feature to the respective Java code.

    ```xtend
    private def compile(Feature f) '''
        private «f.type.fullyQualifiedName» «f.name»;
        
        public «f.type.fullyQualifiedName» get«f.name.toFirstUpper»() {
            return «f.name»;
        }
        
        public void set«f.name.toFirstUpper»(«f.type.fullyQualifiedName» «f.name») {
            this.«f.name» = «f.name»;
        }
    '''
    ```

    As you can see, there is nothing fancy about this one. Last but not least, we have to make sure that the function is actually used.

    ```xtend
    private def compile(Entity e) '''
        «IF e.eContainer.fullyQualifiedName !== null»
            package «e.eContainer.fullyQualifiedName»;
        «ENDIF»
        
        public class «e.name» «IF e.superType !== null
                »extends «e.superType.fullyQualifiedName» «ENDIF»{
            «FOR f : e.features»
                «f.compile»
            «ENDFOR»
        }
    '''
    ```

The final code generator is listed below. Now you can give it a try! Launch a new Eclipse Application (*Run As &rarr; Eclipse Application* on the Xtext project) and create a *dmodel* file in a Java Project. Eclipse will ask you to turn the Java project into an Xtext project then. Simply agree and create a new source folder *src-gen* in that project. Then you can see how the compiler will pick up your sample *Entities* and generate Java code for them.

```xtend
package org.example.domainmodel.generator

import com.google.inject.Inject
import org.eclipse.emf.ecore.resource.Resource
import org.eclipse.xtext.generator.AbstractGenerator
import org.eclipse.xtext.generator.IFileSystemAccess2
import org.eclipse.xtext.generator.IGeneratorContext
import org.eclipse.xtext.naming.IQualifiedNameProvider
import org.example.domainmodel.domainmodel.Entity
import org.example.domainmodel.domainmodel.Feature

class DomainmodelGenerator extends AbstractGenerator {

    @Inject extension IQualifiedNameProvider

    override void doGenerate(Resource resource, IFileSystemAccess2 fsa, IGeneratorContext context) {
        for (e : resource.allContents.toIterable.filter(Entity)) {
            fsa.generateFile(
                e.fullyQualifiedName.toString("/") + ".java",
                e.compile)
        }
    }

    private def compile(Entity e) '''
        «IF e.eContainer.fullyQualifiedName !== null»
            package «e.eContainer.fullyQualifiedName»;
        «ENDIF»
        
        public class «e.name» «IF e.superType !== null
                »extends «e.superType.fullyQualifiedName» «ENDIF»{
            «FOR f : e.features»
                «f.compile»
            «ENDFOR»
        }
    '''

    private def compile(Feature f) '''
        private «f.type.fullyQualifiedName» «f.name»;
        
        public «f.type.fullyQualifiedName» get«f.name.toFirstUpper»() {
            return «f.name»;
        }
        
        public void set«f.name.toFirstUpper»(«f.type.fullyQualifiedName» «f.name») {
            this.«f.name» = «f.name»;
        }
    '''
}
```

If you want to play around with Xtend, you can try to use the Xtend tutorial which can be materialized into your workspace. Simply choose *New &rarr; Example &rarr; Xtend Examples &rarr; Xtend Introductory Examples* and have a look at the features of Xtend. As a small exercise, you could implement support for the *many* attribute of a *Feature* or enforce naming conventions, e.g. generated field names should start with an underscore.

## Creating Custom Validation Rules {#tutorial-validation}

One of the main advantages of DSLs is the possibility to statically validate domain-specific constraints. Because this is a common use case, Xtext provides a dedicated hook for this kind of validation rules. In this lesson, we want to ensure that the name of an *Entity* starts with an upper-case letter and that all features have distinct names across the inheritance relationship of an *Entity*.

Locate the class *DomainmodelValidator* in the package *org.example.domainmodel.validation* of the language project. Defining the constraint itself is only a matter of a few lines of code:

```java
public static final String INVALID_NAME = "invalidName";

@Check
public void checkNameStartsWithCapital(Entity entity) {
    if (!Character.isUpperCase(entity.getName().charAt(0))) {
        warning("Name should start with a capital",
            DomainmodelPackage.Literals.TYPE__NAME,
            INVALID_NAME);
    }
}
```

Any name for the method will do. The important thing is the [Check]({{site.src.xtext_core}}/org.eclipse.xtext/src/org/eclipse/xtext/validation/Check.java) annotation that advises the Xtext framework to use the method as a validation rule. If the name starts with a lower case letter, a warning will be attached to the name of the *Entity*.

The second validation rule is straight-forward, too. We traverse the inheritance hierarchy of the *Entity* and look for features with equal names.

```java
@Check
public void checkFeatureNameIsUnique(Feature feature) {
    Entity superEntity = ((Entity) feature.eContainer()).getSuperType();
    while (superEntity != null) {
        for (Feature other : superEntity.getFeatures()) {
            if (Objects.equal(feature.getName(), other.getName())) {
                error("Feature names have to be unique", DomainmodelPackage.Literals.FEATURE__NAME);
                return;
            }
        }
        superEntity = superEntity.getSuperType();
    }
}
```

The sibling features that are defined in the same entity are automatically validated by the Xtext framework, thus they do not have to be checked twice. Note that this implementation is not optimal in terms of execution time because the iteration over the features is done for all features of each entity.

You can determine when the `@Check`-annotated methods will be executed with the help of the [CheckType]({{site.src.xtext_core}}/org.eclipse.xtext/src/org/eclipse/xtext/validation/CheckType.java) enum. The default value is `FAST`, i.e. the checks will be executed on editing, saving/building or on request; also available are `NORMAL` (executed on build/save or on request) and `EXPENSIVE` (executed only on request).

```java
@Check(CheckType.NORMAL)
public void checkFeatureNameIsUnique(Feature feature) {
    ...
}
```

## Unit Testing the Language {#tutorial-unit-tests}

Automated tests are crucial for the maintainability and the quality of a software product. That is why it is strongly recommended to write unit tests for your language. The Xtext project wizard creates two test projects for that purpose. These simplify the setup procedure for testing the basic language features and the Eclipse UI integration.

This tutorial is about testing the parser, the linker, the validator and the generator of the *Domainmodel* language. It leverages Xtend to write the test cases.

1. The core of the test infrastructure is the [XtextRunner]({{site.src.xtext_core}}/org.eclipse.xtext.testing/src/org/eclipse/xtext/testing/XtextRunner.java) (for JUnit 4) and the language-specific [IInjectorProvider]({{site.src.xtext_core}}/org.eclipse.xtext.testing/src/org/eclipse/xtext/testing/IInjectorProvider.java). Both have to be provided by means of class annotations. An example test class should have already been generated by the Xtext code generator, named *org.example.domainmodel.tests.DomainmodelParsingTest*:
    
    ```xtend
    @RunWith(XtextRunner)
    @InjectWith(DomainmodelInjectorProvider)
    class DomainmodelParsingTest {
    
        @Inject
        ParseHelper<Domainmodel> parseHelper
    
        @Test 
        def void loadModel() {
            val result = parseHelper.parse('''
                Hello Xtext!
            ''')
            Assert.assertNotNull(result)
            val errors = result.eResource.errors
            Assert.assertTrue('''Unexpected errors: «errors.join(", ")»''', errors.isEmpty)
        }
    
    }
    ```

    *Note*: When using JUnit 5 the [InjectionExtension]({{site.src.xtext_core}}/org.eclipse.xtext.testing/src/org/eclipse/xtext/testing/extensions/InjectionExtension.java) is used instead of the XtextRunner. The Xtext code generator generates the example slightly different, depending on which option you have chosen in the *New Xtext Project* wizard.

1. The utility class [ParseHelper]({{site.src.xtext_core}}/org.eclipse.xtext.testing/src/org/eclipse/xtext/testing/util/ParseHelper.java) allows to parse an arbitrary string into a *Domainmodel*. The model itself can be traversed and checked afterwards. A static import of [Assert]({{site.javadoc.junit}}/org/junit/Assert.html) leads to concise and readable test cases. You can rewrite the generated test case as follows:
    
    ```xtend
    import static org.junit.Assert.*

    ...

        @Test 
        def void parseDomainmodel() {
            val model = parseHelper.parse(
                "entity MyEntity {
                    parent: MyEntity
                }")
            val entity = model.elements.head as Entity
            assertSame(entity, entity.features.head.type)
        }
    ```
2. In addition, the utility class [ValidationTestHelper]({{site.src.xtext_core}}/org.eclipse.xtext.testing/src/org/eclipse/xtext/testing/validation/ValidationTestHelper.java) allows to test the custom validation rules written for the language. Both valid and invalid models can be tested.

    ```xtend
    @Inject ParseHelper<Domainmodel> parseHelper

    @Inject ValidationTestHelper validationTestHelper

    @Test
    def testValidModel() {
        val entity = parseHelper.parse(
            "entity MyEntity {
                parent: MyEntity
            }")
        validationTestHelper.assertNoIssues(entity)
    } 

    @Test
    def testNameStartsWithCapitalWarning() {
        val entity = parseHelper.parse(
            "entity myEntity {
                parent: myEntity
            }")
        validationTestHelper.assertWarning(entity,
            DomainmodelPackage.Literals.ENTITY,
            DomainmodelValidator.INVALID_NAME,
            "Name should start with a capital"
        )
    }
    ```
    
    You can further simplify the code by injecting `ParseHelper` and `ValidationTestHelper` as extensions. This feature of Xtend allows to add new methods to a given type without modifying it. You can read more about extension methods in the [Xtend documentation](https://www.eclipse.org/xtend/documentation/202_xtend_classes_members.html#extension-methods). You can rewrite the code as follows:

    ```xtend
    @Inject extension ParseHelper<Domainmodel>

    @Inject extension ValidationTestHelper

    @Test
    def testValidModel() {
        "entity MyEntity {
        parent: MyEntity
        }".parse.assertNoIssues
    }

    @Test
    def testNameStartsWithCapitalWarning() {
        "entity myEntity {
            parent: myEntity
        }".parse.assertWarning(
            DomainmodelPackage.Literals.ENTITY,
            DomainmodelValidator.INVALID_NAME,
            "Name should start with a capital"
        )
    }
    ```

3. The [CompilationTestHelper]({{site.src.xtext_extras}}/org.eclipse.xtext.xbase.testing/src/org/eclipse/xtext/xbase/testing/CompilationTestHelper.java) utility class comes in handy while unit testing the custom generators:

	```xtend
	@Inject extension CompilationTestHelper

	@Test def test() {
		'''
			datatype String
			
			package my.company.blog {
				entity Blog {
					title: String
				}
			}
		'''.assertCompilesTo('''
			package my.company.blog;

			public class Blog {
				private String title;
				
				public String getTitle() {
					return title;
				}
				
				public void setTitle(String title) {
					this.title = title;
				}
			}
		''')
		}
```

4. After saving the Xtend file, it is time to run the tests. Select *Run As &rarr; JUnit Test* from the editor's context menu. All implemented test cases should succeed.

These tests serve only as a starting point and can be extended to cover the different features of the language. As a small exercise, you could implement e.g. test cases for the `checkFeatureNameIsUnique` validation rule. You can find more test cases in the example projects shipped with the Xtext Framework. Simply go to *File &rarr; New &rarr; Example &rarr; Xtext Examples* to instantiate them into your workspace.

---

**[Next Chapter: Five simple steps to your JVM language](104_jvmdomainmodel.html)**

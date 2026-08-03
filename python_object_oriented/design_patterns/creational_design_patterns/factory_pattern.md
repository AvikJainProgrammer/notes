In the first example, they created an abstract class saing that every document type can create and save. 
Then they make concrete implimentations of those.
Then maybe someone wanted to add more functionality to the documents. The could have added the create_document and process document functions in the abstract parent class but they decided to make a new abstract class. 
One of the function as abstract but all implimentations of the factory return a specific type of documents depending on the factory type. And the more interesting function, the process_document function uses the create and save function. 
we could have simply added the process_document in the abstract document class, but I guess we did not do that for the purpose of extending the code. 

Now the second example seems to be bloody redundant. We create database abastract class to connect, disconnect and query and then we impoliment those function for 3 different databases and the factory simple return on the database object based on a string passed to the function. Its not even manipulating any exsingin db connection funciton, its just create the object of the class and return it. One differece from the document example is that we created multiple individual factories for the document example and created one single factory for the db connection example. 

The pyament processor example is similar its just that it is uinga dictionary for mentaining a key value pair for the object name and class. 
The logging example is similar its just that the concrete classes have different kinds of init statements. The function exposed by the factory class takes in kwargs and passes it onto the constructor
The car example is similar to the logger example with one single factory class with multiple where the factory class function has to get kwargs to pass it to concrete class definations. 
The notification example is not even using the kwargs functionality.

In all except the firste example all examples seem to be factory for the sake of factory. 

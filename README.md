# Rust composition pattern
The composite pattern is a structural design pattern that allows you to compose objects into tree structures and then work with these structures as if they were individual objects.


In Rust, we can implement the composite pattern by defining a Component trait that specifies the common interface for both leaf nodes and composite nodes. Then we define Leaf and Composite structs that implement the Component trait. The Composite struct will hold a vector of child components that can be either Leaf nodes or other Composite nodes.

```
trait Component {
    fn operation(&self);
}

struct Leaf {
    // ...
}

impl Component for Leaf {
    fn operation(&self) {
        // Leaf node operation
    }
}

struct Composite {
    children: Vec<Box<dyn Component>>
}

impl Component for Composite {
    fn operation(&self) {
        for child in &self.children {
            child.operation();
        }
    }
}
```
We can now create tree structures like this:

```
let leaf1 = Leaf {};
let leaf2 = Leaf {};

let composite1 = Composite {
    children: vec![Box::new(leaf1)] 
};

let composite2 = Composite {
    children: vec![
        Box::new(leaf2),
        Box::new(composite1)
    ]
};

composite2.operation(); // Operates leaf2 and leaf1
```

And we can call the operation() method on any node, whether it's a leaf node or a composite node, and the correct implementation will be called.

## Use Case: File System
The composite pattern is useful for representing hierarchical tree structures like a file system. We can have File and Directory structs implement a FileSystemItem trait.
```
trait FileSystemItem {
    fn ls(&self);
    fn mkdir(&self, name: &str);
    fn touch(&self, name: &str);
}

struct File {
    // ...
}

impl FileSystemItem for File {
    fn ls(&self) {
        println!("{}", self.name);
    }

    fn mkdir(&self, _name: &str) {
        panic!("Cannot create directory in file!");
    }

    fn touch(&self, _name: &str) {
        panic!("Cannot create file in file!");
    } 
}

struct Directory {
    children: Vec<Box<dyn FileSystemItem>>,
    name: String
}

impl FileSystemItem for Directory {
    fn ls(&self) {
        println!("{}:", self.name);
        for child in &self.children {
            child.ls();
        }
    }

    fn mkdir(&self, name: &str) {
        self.children.push(Box::new(Directory {
            name: name.to_string(),
            children: vec![]
        }));
    } 

    fn touch(&self, name: &str) {
        self.children.push(Box::new(File {
            name: name.to_string()
        }));
    }
}
```
We can now represent a file system hierarchy like this:

```
let root = Directory {
    name: "/".to_string(),
    children: vec![] 
};

root.mkdir("home");
root.mkdir("etc");
root.touch("configfile");

root.children[0].as_ref().unwrap().mkdir("user1"); 
root.children[0].as_ref().unwrap().touch("file1");

root.ls();
```

This will print:
```
/
home: 
user1:
file1
etc:  
configfile
```

And we can call ls(), mkdir() and touch() on any node in the tree, whether it's a File or a Directory, and the correct implementation will be called.

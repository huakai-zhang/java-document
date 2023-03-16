##XML
对象序列化的一个重要限制是它只是Java的解决方案：只有Java程序才能反序列化这种对象。一种更具有互操作性的解决方案是将数据转换为XML格式，这样可以使其被各种各样的平台和语言使用。
开源XOM类库，Person类有一个getXML()方法，使用XOM来产生被转换过的XML的Element对象的Person数据。还有一个构造器，接受Element并从中抽取恰当的Person数据：
```
public class Person {
    private String first, last;

    public Person(String first, String last) {
        this.first = first;
        this.last = last;
    }

    // Produce an XML Element from this Person object:
    public Element getXML() {
        Element person = new Element("person");
        Element firstName = new Element("first");
        firstName.appendChild(first);
        Element lastName = new Element("last");
        lastName.appendChild(last);
        person.appendChild(firstName);
        person.appendChild(lastName);
        return person;
    }

    // Constructor to restore a Person from an XML Element:
    public Person(Element person) {
        first = person.getFirstChildElement("first").getValue();
        last = person.getFirstChildElement("last").getValue();
    }

    @Override
    public String toString() {
        return first + " " + last;
    }

    // Make it human-readable:
    public static void
    format(OutputStream os, Document doc) throws Exception {
        Serializer serializer = new Serializer(os, "ISO-8859-1");
        serializer.setIndent(4);
        serializer.setMaxLength(60);
        serializer.write(doc);
        serializer.flush();
    }

    public static void main(String[] args) throws Exception {
        List<Person> people = Arrays.asList(
                new Person("Dr. Bunsen", "Honeydew"),
                new Person("Gonzo", "The Great"),
                new Person("Phillip J.", "Fry"));
        System.out.println(people);
        Element root = new Element("people");
        for (Person p : people) {
            root.appendChild(p.getXML());
        }
        Document doc = new Document(root);
        format(System.out, doc);
        format(new BufferedOutputStream(new FileOutputStream(
                "People.xml")), doc);
    }
} /*
[Dr. Bunsen Honeydew, Gonzo The Great, Phillip J. Fry]
<?xml version="1.0" encoding="ISO-8859-1"?>
<people>
    <person>
        <first>Dr. Bunsen</first>
        <last>Honeydew</last>
    </person>
    <person>
        <first>Gonzo</first>
        <last>The Great</last>
    </person>
    <person>
        <first>Phillip J.</first>
        <last>Fry</last>
    </person>
</people>
*/
```
XOM的方法都具有相当的自解释性，可以在XOM文档中找到。XOM还包含一个Serializer类，可以在format()方法中看到它被用来将XML转换为更具可读性的格式。如果调用toXML()，那么所有东西都会混在一起，因此Serializer是一种便利工具。
从XML中反序列化Person：
```
public class People extends ArrayList<Person> {
    public People(String fileName) throws Exception {
        Document doc = new Builder().build(fileName);
        Elements elements = doc.getRootElement().getChildElements();
        for (int i = 0; i < elements.size(); i++) {
            add(new Person(elements.get(i)));
        }
    }

    public static void main(String[] args) throws Exception {
        People p = new People("People.xml");
        System.out.println(p);
    }
} /*
[Dr. Bunsen Honeydew, Gonzo The Great, Phillip J. Fry]
*/
```
People构造器使用XOM的Builder.build()方法打开读取一个文件，而getChildElements()方法产生一个Elements列表。每个Element都表示一个Person对象，因此它可以传递给第二个Person构造器。

##Preferences
Preferences API与对象序列化相比，前者与对象持久性更密切。因为它可以自动存储和读取信息。不过，它只能用于小的受限的数据集合——只能存储基本数据和字符串，并且每个字符串的存储长度不能超过8K。顾名思义，Preferences API用于存储和读取用户的偏好preferences以及程序配置项的设置。
Preferences是一个键-值集合，存储在一个节点层次结构中。
```
public class PreferencesDemo {
    public static void main(String[] args) throws Exception {
        Preferences prefs = Preferences
                .userNodeForPackage(PreferencesDemo.class);
        prefs.put("Location", "Oz");
        prefs.put("Footwear", "Ruby Slippers");
        prefs.putInt("Companions", 4);
        prefs.putBoolean("Are there witches?", true);
        int usageCount = prefs.getInt("UsageCount", 0);
        usageCount++;
        prefs.putInt("UsageCount", usageCount);
        for (String key : prefs.keys()) {
            print(key + ": " + prefs.get(key, null));
        }
        // You must always provide a default value:
        print("How many companions does Dorothy have? " +
                prefs.getInt("Companions", 0));
    }
} /*
Location: Oz
Footwear: Ruby Slippers
Companions: 4
Are there witches?: true
UsageCount: 53
How many companions does Dorothy have? 4
*/
```
这里用的是userNodeForPackage(),但我们也可以选择用systemNodeForPackage();虽然可以任意选择，但最好将“user” 用于个别用户的偏好，将“system”用于通用的安装配置。因为main()是静态的，因此PreferencesDemo.class可以用来标识节点; 但是在非静态方法内部，我们通常使用getClass()。尽管我们不一定非要把当前的类作为节点标识符，但这仍不失为一种很有用的方法。
一旦我们创建了节点，就可以用它来加载或者读取数据了。在这个例子中，向节点载入了各种不同类型的数据项，然后获取其keys()。它们是以String[]的形式返回的，如果你习惯于keys()属于集合类库，那么这个返回结果可能并不是你所期望的。注意get()的第二个参数，如果某个关键字下没有任何条目，那么这个参数就是所产生的默认值。当在一个关键字集合内迭代时，我们总要确信条目是存在的，因此用null作为默认值是安全的，但是通常我们会获得一个具名的关键字，就像下面这条语句:
  prefs.getInt("Companions". 0)):
在通常情况下，我们希望提供一个合理的默认值。实际上，典型的习惯用法可见下面几行:
  int usageCount = prefs.getInt("UsageCount", 0);usageCount++;
  prefs.putInt("UsageCount", usageCount);
这样，在我们第一次运行程序时，UsageCount的值是0, 但在随后引用中，它将会是非零值。
在我们运行PreferencesDemo.java时，会发现每次运行程序时，UsageCount的值都会增加1，但是数据存储到哪里了呢?在程序第一次运行之后，并没有出现任何本地文件。Preferences API利用合适的系统资源完成了这个任务，并且这些资源会随操作系统不同而不同。例如在Windows里，就使用注册表(因为它已经有“键值对”这样的节点对层次结构了)。但是最重要的一点是，它已经神奇般地为我们存储了信息，所以我们不必担心不同的操作系统是怎么运作的。

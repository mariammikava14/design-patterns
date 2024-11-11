# 1) Strategy Pattern
```cpp
#include <iostream>
#include <memory>
#include <string>

// Strategy ინტერფეისი
class Strategy {
public:
    virtual int execute(int a, int b) const = 0;
    virtual ~Strategy() = default;
};

// კონკრეტული strategy დასამატებლად
class ConcreteStrategyAdd : public Strategy {
public:
    int execute(int a, int b) const override {
        return a + b;
    }
};

// კონკრეტული strategy გამოსაკლებად
class ConcreteStrategySubtract : public Strategy {
public:
    int execute(int a, int b) const override {
        return a - b;
    }
};

// კონკრეტული strategy გასამრავლებლად
class ConcreteStrategyMultiply : public Strategy {
public:
    int execute(int a, int b) const override {
        return a * b;
    }
};

// Context კლასი
class Context {
private:
    std::unique_ptr<Strategy> strategy;

public:
    void setStrategy(std::unique_ptr<Strategy> newStrategy) {
        strategy = std::move(newStrategy);
    }

    int executeStrategy(int a, int b) const {
        if (strategy) {
            return strategy->execute(a, b);
        }
        std::cerr << "Strategy not set." << std::endl;
        return 0;
    }
};

// ExampleApplication კლასი
class ExampleApplication {
public:
    void main() {
        Context context;
        int a, b;
        std::string action;

        std::cout << "Enter first number: ";
        std::cin >> a;
        std::cout << "Enter second number: ";
        std::cin >> b;
        std::cout << "Choose operation (add, subtract, multiply): ";
        std::cin >> action;

        if (action == "add") {
            context.setStrategy(std::make_unique<ConcreteStrategyAdd>());
        } else if (action == "subtract") {
            context.setStrategy(std::make_unique<ConcreteStrategySubtract>());
        } else if (action == "multiply") {
            context.setStrategy(std::make_unique<ConcreteStrategyMultiply>());
        } else {
            std::cout << "Invalid action" << std::endl;
            return;
        }

        int result = context.executeStrategy(a, b);
        std::cout << "Result: " << result << std::endl;
    }
};

int main() {
    ExampleApplication app;
    app.main();
    return 0;
}
```
- This is the Strategy Interface. It declares a virtual method, execute(int a, int b), which derived classes will override.
- The = 0 part makes execute a pure virtual function, making Strategy an abstract class.
- The ~Strategy() = default; provides a virtual destructor, ensuring proper cleanup of derived classes.
- ეს არის სტრატეგიის ინტერფეისი. იგი აცხადებს ვირტუალურ მეთოდს, execute(int a, int b), რომელიც გამოყვანილი კლასები გადაიჭრება.
- = 0 ნაწილი ახორციელებს სუფთა ვირტუალურ ფუნქციას, რაც სტრატეგიას აბსტრაქტულ კლასად აქცევს.
- ~სტრატეგია() = ნაგულისხმევი; უზრუნველყოფს ვირტუალურ დესტრუქტორს, რომელიც უზრუნველყოფს მიღებული კლასების სათანადო გასუფთავებას.
- setStrategy მეთოდი საშუალებას გაძლევთ შეცვალოთ სტრატეგია გაშვების დროს სტრატეგიის ობიექტის საკუთრებაში მიღების გზით std::unique_ptr-ის გამოყენებით.
 executeStrategy მეთოდი უწოდებს მიმდინარე სტრატეგიის შესრულების მეთოდს. თუ სტრატეგია არ არის დაყენებული, ის გამოსცემს შეცდომას და აბრუნებს 0-ს.
- ExampleApplication ურთიერთქმედებს მომხმარებელთან, რათა მიიღოს შეყვანის მნიშვნელობები და სასურველი ოპერაცია (შეკრება, გამოკლება ან გამრავლება).
მომხმარებლის შეყვანიდან გამომდინარე, ის ადგენს შესაბამის სტრატეგიას კონტექსტში.
საბოლოოდ, ის ახორციელებს არჩეულ სტრატეგიას და გამოაქვს შედეგი.
- როგორ მუშაობს

 მომხმარებლის ურთიერთქმედება: ExampleApplication სთხოვს მომხმარებელს ორ რიცხვს და ოპერაციას, რომლის შესრულებაც სურს.
 სტრატეგიის დაყენება: მომხმარებლის არჩევანის საფუძველზე, შესაბამისი სტრატეგია (დამატება, გამოკლება ან გამრავლება) დაყენებულია კონტექსტში.
 შესრულება: შემდეგ კონტექსტი ასრულებს ოპერაციას სტრატეგიის შესრულების მეთოდის გამოძახებით და აჩვენებს შედეგს.

სტრატეგიის ნიმუშის გამოყენების უპირატესობები

 მოქნილობა: თქვენ შეგიძლიათ მარტივად დაამატოთ ახალი სტრატეგიები არსებული კოდის შეცვლის გარეშე.
 გამოყოფა: ალგორითმი (სტრატეგია) გამოყოფილია კონტექსტიდან, რაც საშუალებას იძლევა უკეთესი კოდის ორგანიზება.
 ურთიერთშემცვლელობა: სტრატეგიები შეიძლება შეიცვალოს დინამიურად მუშაობის დროს.
 
---

# 2) Observer Pattern
```cpp
#include <iostream>
#include <unordered_map>
#include <vector>
#include <memory>
#include <string>
#include <algorithm>

// EventListener - ის ინტერფეისი
class EventListener {
public:
    virtual void update(const std::string& filename) = 0;
};

// EventManager კლასი subscriptions - ისთვის
class EventManager {
private:
    std::unordered_map<std::string, std::vector<std::shared_ptr<EventListener>>> listeners;

public:
    void subscribe(const std::string& eventType, std::shared_ptr<EventListener> listener) {
        listeners[eventType].push_back(listener);
    }

    void unsubscribe(const std::string& eventType, std::shared_ptr<EventListener> listener) {
        auto& eventListeners = listeners[eventType];
        eventListeners.erase(
            std::remove(eventListeners.begin(), eventListeners.end(), listener),
            eventListeners.end()
        );
    }

    void notify(const std::string& eventType, const std::string& data) {
        for (const auto& listener : listeners[eventType]) {
            listener->update(data);
        }
    }
};

// Editor კლასი (publisher)
class Editor {
public:
    EventManager events;
    std::string file;

    void openFile(const std::string& path) {
        file = path;
        events.notify("open", file);
    }

    void saveFile() {
        // ინახავს ფაილის ოპერაციას
        events.notify("save", file);
    }
};

// კონკრეტული subscriber - LoggingListener
class LoggingListener : public EventListener {
private:
    std::string log;
    std::string message;

public:
    LoggingListener(const std::string& logFilename, const std::string& msg)
        : log(logFilename), message(msg) {}

    void update(const std::string& filename) override {
        std::cout << message << " " << filename << std::endl;
        // log file - ის დაწერა
    }
};

// კონკრეტული subscriber - EmailAlertsListener
class EmailAlertsListener : public EventListener {
private:
    std::string email;
    std::string message;

public:
    EmailAlertsListener(const std::string& emailAddr, const std::string& msg)
        : email(emailAddr), message(msg) {}

    void update(const std::string& filename) override {
        std::cout << "Sending email to " << email << ": " << message << " " << filename << std::endl;
        // email ის გაგზავნა
    }
};

// Application კლასი, რომ გააერთიანოს publishers და subscribers
class Application {
public:
    void config() {
        Editor editor;

        auto logger = std::make_shared<LoggingListener>(
            "/path/to/log.txt",
            "Someone has opened the file:"
        );
        editor.events.subscribe("open", logger);

        auto emailAlerts = std::make_shared<EmailAlertsListener>(
            "admin@example.com",
            "Someone has changed the file:"
        );
        editor.events.subscribe("save", emailAlerts);

        // გამოყენება
        editor.openFile("example.txt");
        editor.saveFile();
    }
};

int main() {
    Application app;
    app.config();

    return 0;
}
```
- მიზანი: ეს არის EventListener ინტერფეისი, რომელიც აცხადებს განახლების მეთოდს. ნებისმიერი კლასი, რომელიც გამოიწერს მოვლენებს, განახორციელებს ამ ინტერფეისს და განსაზღვრავს მის განახლების მეთოდს კონკრეტული მოვლენების დასამუშავებლად.
რატომ: ეს უზრუნველყოფს, რომ ყველა აბონენტს (როგორიცაა LoggingListener და EmailAlertsListener) ჰქონდეს საერთო ინტერფეისი, რაც მათ საშუალებას აძლევს მიიღონ შეტყობინებები EventManager-ისგან.
- მიზანი: EventManager მართავს მსმენელთა (აბონენტების) გამოწერებს და აცნობებს მათ მოვლენებს.
ატრიბუტები:

 მსმენელები: რუკა, სადაც თითოეული ღონისძიების ტიპი (მაგ., „გახსნა“ ან „შენახვა“) ასოცირდება მსმენელთა სიასთან, რომლებსაც უნდა ეცნობოს ამ მოვლენის შესახებ.

მეთოდები:

 გამოწერა: ამატებს მსმენელს მსმენელთა სიაში კონკრეტული მოვლენის ტიპისთვის.
 გამოწერის გაუქმება: ამოიღებს მსმენელს სიიდან კონკრეტული მოვლენის ტიპისთვის.
 notify: აცნობებს ყველა მსმენელს, რომელიც გამოწერილია მოცემული მოვლენის ტიპზე მათი განახლების მეთოდის დარეკვით.

 -მიზანი: ამ შაბლონში რედაქტორი მოქმედებს როგორც გამომცემელი. მას აქვს ორი ძირითადი მოქმედება: ფაილის გახსნა და შენახვა.
ატრიბუტები:

 მოვლენები: EventManager-ის მაგალითი, რომელიც ამუშავებს აბონენტებს და აცნობებს მათ ნებისმიერი მოვლენის შესახებ.
 ფაილი: მიმდინარე ფაილი იმართება.

მეთოდები:

 openFile: ხსნის ფაილს და აცნობებს მსმენელს "ღია" მოვლენის შესახებ.
 saveFile: ინახავს ფაილს და აცნობებს მსმენელს "შენახვის" მოვლენის შესახებ.

 - მიზანი: LoggingListener არის კონკრეტული აბონენტი, რომელიც წერს შეტყობინებას, როდესაც ეცნობება მოვლენის შესახებ.
ატრიბუტები:

 ჟურნალი: ინახავს ჟურნალის ფაილის გზას (აქ არ გამოიყენება, მაგრამ შეიძლება გამოყენებულ იქნას ჟურნალის ფაილში შეტყობინებების ჩასაწერად).
 შეტყობინება: შეტყობინება, რომლის საშუალებითაც შეგიძლიათ დარეგისტრირდეთ, როდესაც ხდება მოვლენა.

მეთოდები:

 განახლება: ბეჭდავს ჟურნალის შეტყობინებას, როდესაც განახლების მეთოდი გამოიძახება EventManager-ის მიერ.

 - მიზანი: EmailAlertsListener არის კიდევ ერთი კონკრეტული აბონენტი, რომელიც სიმულაციას უკეთებს ელ.ფოსტის გაფრთხილების გაგზავნას მოვლენის შესახებ შეტყობინებისას.
ატრიბუტები:

 ელფოსტა: მიმღების ელფოსტის მისამართი.
 შეტყობინება: შეტყობინება, რომელიც უნდა ჩართოთ ელფოსტაში.

მეთოდები:

 განახლება: ბეჭდავს შეტყობინებას ელ.ფოსტის გაგზავნის სიმულაციისთვის, როდესაც ხდება მოვლენა.

 - მიზანი: აპლიკაციის კლასი აკონფიგურირებს რედაქტორს თავის აბონენტებთან ერთად, აჩვენებს, თუ როგორ გამოიყენოთ Observer ნიმუში.
 მეთოდები:
 config: ქმნის რედაქტორის მაგალითს და ორ მსმენელს (LoggingListener და EmailAlertsListener). შემდეგ ის აწერს მათ "ღია" და "შენახვა" მოვლენებს, შესაბამისად, და ახდენს რედაქტორის ფაილის გახსნისა და შენახვის სიმულაციას.
- მიზანი: პროგრამის შესვლის წერტილი. ის ქმნის აპლიკაციის მაგალითს, აკონფიგურირებს და აწარმოებს მას.

რეზიუმე

 EventListener ინტერფეისი: აცხადებს განახლების მეთოდს აბონენტებისთვის.
 EventManager: მართავს გამოწერებს და აცნობებს მსმენელებს მოვლენების შესახებ.
 რედაქტორი: მოქმედებს როგორც გამომცემელი, აცნობებს EventManager-ს კონკრეტული ქმედებების დროს.
 კონკრეტული აბონენტები (LoggingListener და EmailAlertsListener): განახლების მეთოდის დანერგვა, შეტყობინებების დამუშავება კონკრეტული გზებით (ლოგირება ან ელფოსტა).
 აპლიკაცია: აკონფიგურირებს რედაქტორს თავის აბონენტებთან ერთად, ადგენს მოვლენებზე ორიენტირებულ ქცევას.

---
# Decorator 
```cpp
#include <iostream>
#include <memory>
#include <string>

// DataSource ინტერფეისი
class DataSource {
public:
    virtual void writeData(const std::string& data) = 0;
    virtual std::string readData() const = 0;
    virtual ~DataSource() = default;
};

// კონკრეტული კომპონენტი: FileDataSource
class FileDataSource : public DataSource {
private:
    std::string filename;

public:
    FileDataSource(const std::string& filename) : filename(filename) {}

    void writeData(const std::string& data) override {
        std::cout << "Writing data to file: " << data << std::endl;
        //  ვწერთ მონაცემებს ფაილში
    }

    std::string readData() const override {
        std::cout << "Reading data from file." << std::endl;
        // ვკითხულობთ მონაცემებს ფაილიდან
        return "file data";
    }
};

// ფუძე decorator კლასი
class DataSourceDecorator : public DataSource {
protected:
    std::unique_ptr<DataSource> wrappee;

public:
    DataSourceDecorator(std::unique_ptr<DataSource> source) : wrappee(std::move(source)) {}

    void writeData(const std::string& data) override {
        wrappee->writeData(data);
    }

    std::string readData() const override {
        return wrappee->readData();
    }
};

// კონკრეტული decorator: EncryptionDecorator
class EncryptionDecorator : public DataSourceDecorator {
public:
    EncryptionDecorator(std::unique_ptr<DataSource> source) : DataSourceDecorator(std::move(source)) {}

    void writeData(const std::string& data) override {
        std::string encryptedData = "Encrypted(" + data + ")";
        std::cout << "Encrypting data and passing it to the wrappee." << std::endl;
        wrappee->writeData(encryptedData);
    }

    std::string readData() const override {
        std::string data = wrappee->readData();
        std::string decryptedData = "Decrypted(" + data + ")";
        std::cout << "Decrypting data from wrappee." << std::endl;
        return decryptedData;
    }
};

// კონკრეტული decorator: CompressionDecorator
class CompressionDecorator : public DataSourceDecorator {
public:
    CompressionDecorator(std::unique_ptr<DataSource> source) : DataSourceDecorator(std::move(source)) {}

    void writeData(const std::string& data) override {
        std::string compressedData = "Compressed(" + data + ")";
        std::cout << "Compressing data and passing it to the wrappee." << std::endl;
        wrappee->writeData(compressedData);
    }

    std::string readData() const override {
        std::string data = wrappee->readData();
        std::string decompressedData = "Decompressed(" + data + ")";
        std::cout << "Decompressing data from wrappee." << std::endl;
        return decompressedData;
    }
};

// გამოყენების მაგალითი - class: Application
class Application {
public:
    void dumbUsageExample() {
        std::unique_ptr<DataSource> source = std::make_unique<FileDataSource>("somefile.dat");

        source->writeData("SalaryRecords");
        // მონაცემების ჩაწერა

        source = std::make_unique<CompressionDecorator>(std::move(source));
        source->writeData("SalaryRecords");
        // შევიწროებული მონაცემების ჩაწერა

        source = std::make_unique<EncryptionDecorator>(std::move(source));
        source->writeData("SalaryRecords");
        // შეკუმშული და დაშიფრული მონაცემების ჩაწერა
    }
};

// კლიენტის კოდი რომელიც იყენება გარე მონაცემებს - SalaryManager
class SalaryManager {
private:
    std::unique_ptr<DataSource> source;

public:
    SalaryManager(std::unique_ptr<DataSource> src) : source(std::move(src)) {}

    std::string load() {
        return source->readData();
    }

    void save(const std::string& data) {
        source->writeData(data);
    }
};

// ApplicationConfigurator კლასი
class ApplicationConfigurator {
public:
    void configurationExample(bool enabledEncryption, bool enabledCompression) {
        std::unique_ptr<DataSource> source = std::make_unique<FileDataSource>("salary.dat");

        if (enabledEncryption) {
            source = std::make_unique<EncryptionDecorator>(std::move(source));
        }
        if (enabledCompression) {
            source = std::make_unique<CompressionDecorator>(std::move(source));
        }

        SalaryManager manager(std::move(source));
        std::string salary = manager.load();
        std::cout << "Loaded data: " << salary << std::endl;
    }
};

int main() {
    Application app;
    app.dumbUsageExample();

    ApplicationConfigurator configurator;
    configurator.configurationExample(true, true);

    return 0;
}
```
- This code demonstrates the Decorator Pattern in C++. The Decorator Pattern is used to add behavior dynamically to objects without modifying their underlying structure or classes. In this example, the DataSource interface represents the core functionality (reading and writing data), and decorators are used to add additional behaviors like encryption and compression.

- ეს კოდი აჩვენებს დეკორატორის შაბლონს C++-ში. Decorator Pattern გამოიყენება ობიექტების ქცევის დინამიურად დასამატებლად მათი ძირითადი სტრუქტურის ან კლასების შეცვლის გარეშე. ამ მაგალითში, DataSource ინტერფეისი წარმოადგენს ძირითად ფუნქციონირებას (მონაცემების კითხვა და ჩაწერა), ხოლო დეკორატორები გამოიყენება დამატებითი ქცევების დასამატებლად, როგორიცაა დაშიფვრა და შეკუმშვა.
- მიზანი: DataSource განსაზღვრავს ინტერფეისს მონაცემების წაკითხვისა და ჩაწერისთვის. ყველა კონკრეტული განხორციელება და დეკორატორი განახორციელებს ამ ინტერფეისს.
რატომ: დეკორატორის ნიმუში მოითხოვს ინტერფეისს ან საბაზისო კლასს, რომელსაც ორივე ძირითადი კომპონენტი და დეკორატორები ახორციელებენ, რათა მათი გამოყენება ურთიერთშენაცვლებით იყოს შესაძლებელი.
- მიზანი: FileDataSource არის DataSource-ის კონკრეტული განხორციელება, რომელიც პირდაპირ კითხულობს და წერს ფაილიდან.
როლი დეკორატორის ნიმუშში: ეს კლასი არის ძირითადი კომპონენტი, რომელიც ასრულებს მონაცემთა რეალურ დამუშავებას. დეკორატორები ამ კომპონენტს „გაახვევენ“ ქცევის დასამატებლად მისი შეცვლის გარეშე.
- მიზანი: DataSourceDecorator არის საბაზისო დეკორატორის კლასი, რომელიც ასევე ახორციელებს DataSource ინტერფეისს და აქვს მაჩვენებელი DataSource ობიექტზე ("wrappee").
 როგორ მუშაობს: ეს კლასი გადასცემს სამუშაოს შეფუთულ DataSource ობიექტს. თავისთავად, ის არ ცვლის რაიმე ქცევას, მაგრამ ემსახურება როგორც სხვა დეკორატორების გაფართოების საფუძველს.
 როლი დეკორატორის ნიმუშში: ეს არის საბაზისო დეკორატორი, რომელიც საშუალებას აძლევს დამატებით ქცევებს დაემატოს თანმიმდევრული ინტერფეისის შენარჩუნებისას.

4. ბეტონის დეკორატორები: EncryptionDecorator და CompressionDecorator

 ეს კლასები მემკვიდრეობით მიიღება DataSourceDecorator-ისგან და ამატებენ დამატებით ფუნქციებს (დაშიფვრა და შეკუმშვა) მონაცემებს.

 - რეზიუმე: რატომ არის ეს დეკორატორის ნიმუში

 დინამიური ქცევის მოდიფიკაცია: CompressionDecorator და EncryptionDecorator ამატებენ დამატებით ფუნქციებს (შეკუმშვა და დაშიფვრა) FileDataSource-ის შეცვლის გარეშე.
 მოქნილი დეკორაცია: დეკორატორების დაწყობით, ჩვენ შეგვიძლია გამოვიყენოთ მრავალი ქცევა დინამიურად. კლიენტის კოდს არ სჭირდება დეკორატორების დეტალების ცოდნა, მხოლოდ DataSource ინტერფეისი.
 თანმიმდევრულობა: თითოეული დეკორატორი და FileDataSource იზიარებენ ერთსა და იმავე ინტერფეისს (DataSource), რაც მათ საშუალებას აძლევს გამოიყენონ ურთიერთშემცვლელნი და ადვილად შეიფუთონ.

ეს სტრუქტურა აერთიანებს დამატებით ქცევებს ცალკეულ კლასებში და უზრუნველყოფს მოქნილობას სისტემის გაფართოვებისთვის ახალი დეკორატორებით, თუ მომავალში საჭირო იქნება მეტი ქცევა (როგორიცაა აღრიცხვა, ქეშირება და ა.შ.).

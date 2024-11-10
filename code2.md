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

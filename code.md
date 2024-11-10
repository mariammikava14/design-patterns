# 1) Strategy Pattern (composition over inheritance)

```cpp
#include <iostream>

class SocialMediaStrategy {
public:
    virtual void post() = 0;
    virtual ~SocialMediaStrategy() {}
};

class LinkedInStrategy : public SocialMediaStrategy {
public:
    void post() {
        std::cout << "LinkedInStrategy\n";
    }
};

class TwitterStrategy : public SocialMediaStrategy {
public:
    void post() {
        std::cout << "TwitterStrategy\n";
    }
};

class User {
private:
    SocialMediaStrategy* socialMedia;

public:
    User(SocialMediaStrategy* socialMedia) {
        this->socialMedia = socialMedia;
    }

    void postOnSocialMedia() {
        socialMedia->post();
    }

    void setSocialMediaStrategy(SocialMediaStrategy* socialMedia) {
        this->socialMedia = socialMedia;
    }
};

int main() {
    User* user = new User(new TwitterStrategy());
    user->postOnSocialMedia();

    user->setSocialMediaStrategy(new LinkedInStrategy);
    user->postOnSocialMedia();
}
```
# 2) Observer Pattern code
```cpp
#include <iostream>
#include <vector>
#include <string>
#include <algorithm>

class IObserver{
  public:
    virtual void update() {};
    virtual ~IObserver() {}
};

class IObservable{
  public:
    virtual void add(IObserver* observer) {};
    virtual void remove(IObserver* observer) {};
    virtual void notify() {};
};

class WeatherStation : public IObservable {
  private:
    std::vector<IObserver*> subscribers;
  public:
    void add(IObserver* observer){
      subscribers.push_back(observer);
    }

    void remove(IObserver* observer){
      auto it = std::find(subscribers.begin(), subscribers.end(), observer);
      if (it != subscribers.end())
        subscribers.erase(it);
    }
    
    void notify(){
      for(auto e : subscribers){
        e->update();
      }
    }

    std::string getWeather(){
      return "Weather good";
    }

    ~WeatherStation() {
      for(auto e : subscribers){
        e = nullptr;
        delete e;
      }
      subscribers.clear();
    }
};

class Computer : public IObserver{
  private:
    WeatherStation* station;
    std::string name;
  public:
    Computer(WeatherStation* station){
      this->station = station;
    }

    void setName(std::string name){
      this->name = name;
    }

    void update(){
      std::cout << name << ' ' << station->getWeather() << '\n';
    }
    
    ~Computer() {
      std::cout << name << " deleted.\n";
      station = nullptr;
      delete station;
    }
};

int main(){
  WeatherStation station;

  Computer c1(&station);
  c1.setName("computer1");
  Computer c2(&station);
  c2.setName("computer2");
  Computer c3(&station);
  c3.setName("computer3");

  station.add(&c1);
  station.add(&c2);

  station.notify();
}
```
# 3) Decorator Pattern code
```cpp
#include <iostream>
#include <string>

//Base Class
class Beverage {
  public:
    virtual std::string getDescription() = 0;
    virtual float getCost() = 0;
    virtual ~Beverage() {std::cout<<"\nBeverage destructed.\n";}
};

//Class for decorative stuff, like milk.
//Sure it has nothing inside, but it's a nice way to separate something like Americano and Milk.
class CondinentDecorator : public Beverage{ };

class Americano : public Beverage {
  private:
    float cost = 1.69;
  public:
    std::string getDescription() { return "Americano"; }
    float getCost() { return cost; }
};

class Milk : public CondinentDecorator {
  private:
    float cost = 0.35;
    Beverage* beverage;
  public:
    Milk(Beverage* beverage){
      this->beverage = beverage;
    }

    std::string getDescription() { return beverage->getDescription() + " with Milk "; }
    float getCost(){
      return beverage->getCost() + cost;
    }
};

int main(){
  Beverage* coffee = new Milk(new Milk(new Americano()));
  std::cout << coffee->getCost() << coffee->getDescription();

  delete coffee;
  coffee = nullptr;
}
```

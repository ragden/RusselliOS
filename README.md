# Russell iOS
Russell its a Framework to make easier use VIPER pattern in iOS apps. This framework propose one way to use this architecture. You can read more information about VIPER in http://www.objc.io/issue-13/viper.html

# What is VIPER?
VIPER is an application of Clean Architecture to iOS apps. The word VIPER is a backronym for View, Interactor, Presenter, Entity, and Routing. Clean Architecture divides an app’s logical structure into distinct layers of responsibility. This makes it easier to isolate dependencies (e.g. your database) and to test the interactions at the boundaries between layers.

# Main Parts of VIPER
The main parts of VIPER are:

View: displays what it is told to by the Presenter and relays user input back to the Presenter.
Interactor: contains the business logic as specified by a use case.
Presenter: contains view logic for preparing content for display (as received from the Interactor) and for reacting to user inputs (by requesting new data from the Interactor).
Entity: contains basic model objects used by the Interactor.
Routing: contains navigation logic for describing which screens are shown in which order.
This separation also conforms to the Single Responsibility Principle. The Interactor is responsible to the business analyst, the Presenter represents the interaction designer, and the View is responsible to the visual designer.

# Interactor
An Interactor represents a single use case in the app. It contains the business logic to manipulate model objects (Entities) to carry out a specific task. The work done in an Interactor should be independent of any UI. The same Interactor could be used in an iOS app or an OS X app.

Because the Interactor is a PONSO (Plain Old NSObject) that primarily contains logic, it is easy to develop using TDD.

The primary use case for the sample app is to show the user any upcoming to-do items (i.e. anything due by the end of next week). The business logic for this use case is to find any to-do items due between today and the end of next week and assign a relative due date: today, tomorrow, later this week, or next week.

Below is the corresponding method from VTDListInteractor:
```objc
- (void)findUpcomingItems
{
    __weak typeof(self) welf = self;
    NSDate* today = [self.clock today];
    NSDate* endOfNextWeek = [[NSCalendar currentCalendar] dateForEndOfFollowingWeekWithDate:today];
    [self.dataManager todoItemsBetweenStartDate:today endDate:endOfNextWeek completionBlock:^(NSArray* todoItems) {
        [welf.output foundUpcomingItems:[welf upcomingItemsFromToDoItems:todoItems]];
    }];
}
```

# Entity
Entities are the model objects manipulated by an Interactor. Entities are only manipulated by the Interactor. The Interactor never passes entities to the presentation layer (i.e. Presenter).

Entities also tend to be PONSOs. If you are using Core Data, you will want your managed objects to remain behind your data layer. Interactors should not work with NSManagedObjects.

Here is the Entity for our to-do item:

```objc
@interface VTDTodoItem : NSObject

@property (nonatomic, strong)   NSDate*     dueDate;
@property (nonatomic, copy)     NSString*   name;

+ (instancetype)todoItemWithDueDate:(NSDate*)dueDate name:(NSString*)name;

@end
```

Don’t be surprised if your entities are just data structures. Any application-dependent logic will most likely be in an Interactor.

# Presenter
The Presenter is a PONSO that mainly consists of logic to drive the UI. It knows when to present the user interface. It gathers input from user interactions so it can update the UI and send requests to an Interactor.

When the user taps the + button to add a new to-do item, addNewEntry gets called. For this action, the Presenter asks the wireframe to present the UI for adding a new item:

```objc
- (void)addNewEntry
{
    [self.listWireframe presentAddInterface];
}
The Presenter also receives results from an Interactor and converts the results into a form that is efficient to display in a View.

Below is the method that receives upcoming items from the Interactor. It will process the data and determine what to show to the user:

- (void)foundUpcomingItems:(NSArray*)upcomingItems
{
    if ([upcomingItems count] == 0)
    {
        [self.userInterface showNoContentMessage];
    }
    else
    {
        [self updateUserInterfaceWithUpcomingItems:upcomingItems];
    }
}
````

Entities are never passed from the Interactor to the Presenter. Instead, simple data structures that have no behavior are passed from the Interactor to the Presenter. This prevents any ‘real work’ from being done in the Presenter. The Presenter can only prepare the data for display in the View.

# View
The View is passive. It waits for the Presenter to give it content to display; it never asks the Presenter for data. Methods defined for a View (e.g. LoginView for a login screen) should allow a Presenter to communicate at a higher level of abstraction, expressed in terms of its content, and not how that content is to be displayed. The Presenter does not know about the existence of UILabel, UIButton, etc. The Presenter only knows about the content it maintains and when it should be displayed. It is up to the View to determine how the content is displayed.

The View is an abstract interface, defined in Objective-C with a protocol. A UIViewController or one of its subclasses will implement the View protocol. For example, the ‘add’ screen from our example has the following interface:

```objc
@protocol VTDAddViewInterface <NSObject>

- (void)setEntryName:(NSString *)name;
- (void)setEntryDueDate:(NSDate *)date;

@end
```

Views and view controllers also handle user interaction and input. It’s easy to understand why view controllers usually become so large, since they are the easiest place to handle this input to perform some action. To keep our view controllers lean, we need to give them a way to inform interested parties when a user takes certain actions. The view controller shouldn’t be making decisions based on these actions, but it should pass these events along to something that can.

In our example, Add View Controller has an event handler property that conforms to the following interface:

```objc
@protocol VTDAddModuleInterface <NSObject>

- (void)cancelAddAction;
- (void)saveAddActionWithName:(NSString *)name dueDate:(NSDate *)dueDate

@end
````

When the user taps on the cancel button, the view controller tells this event handler that the user has indicated that it should cancel the add action. That way, the event handler can take care of dismissing the add view controller and telling the list view to update.

The boundary between the View and the Presenter is also a great place for ReactiveCocoa. In this example, the view controller could also provide methods to return signals that represent button actions. This would allow the Presenter to easily respond to those signals without breaking separation of responsibilities.

#Routing

Routes from one screen to another are defined in the wireframes created by an interaction designer. In VIPER, the responsibility for Routing is shared between two objects: the Presenter, and the wireframe. A wireframe object owns the UIWindow, UINavigationController, UIViewController, etc. It is responsible for creating a View/ViewController and installing it in the window.

Since the Presenter contains the logic to react to user inputs, it is the Presenter that knows when to navigate to another screen, and which screen to navigate to. Meanwhile, the wireframe knows how to navigate. So, the Presenter will use the wireframe to perform the navigation. Together, they describe a route from one screen to the next.

The wireframe is also an obvious place to handle navigation transition animations. Take a look at this example from the add wireframe:

```objc
@implementation VTDAddWireframe

- (void)presentAddInterfaceFromViewController:(UIViewController *)viewController 
{
    VTDAddViewController *addViewController = [self addViewController];
    addViewController.eventHandler = self.addPresenter;
    addViewController.modalPresentationStyle = UIModalPresentationCustom;
    addViewController.transitioningDelegate = self;

    [viewController presentViewController:addViewController animated:YES completion:nil];

    self.presentedViewController = viewController;
}

#pragma mark - UIViewControllerTransitioningDelegate Methods

- (id<UIViewControllerAnimatedTransitioning>)animationControllerForDismissedController:(UIViewController *)dismissed 
{
    return [[VTDAddDismissalTransition alloc] init];
}

- (id<UIViewControllerAnimatedTransitioning>)animationControllerForPresentedController:(UIViewController *)presented
                                                                  presentingController:(UIViewController *)presenting
                                                                      sourceController:(UIViewController *)source 
{
    return [[VTDAddPresentationTransition alloc] init];
}

@end
````

The app is using a custom view controller transition to present the add view controller. Since the wireframe is responsible for performing the transition, it becomes the transitioning delegate for the add view controller and can return the appropriate transition animations.

# RusselliOS
Russell its a Framework to make easier use VIPER pattern in iOS apps. 

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


This framework propose one way to use this architecture. You can read more information about VIPER in http://www.objc.io/issue-13/viper.html

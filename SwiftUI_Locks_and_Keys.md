# Locks and Keys in SwiftUI

I always wanted to try the "Locks and Keys" principle described in this old article:

https://www.swiftbysundell.com/articles/managing-objects-using-locks-and-keys-in-swift/

But all the boilerplate with factories has been discouraging me so far.

Today I gave it a go and managed to apply it in a very simple and lightweight way into SwiftUI and I want to share it with you.

First, what problem does the "Locks and Keys" principle try to solve?

We often need to work with optionals which are conceptually non-optional but technically optional.
One example is the logged in user. We have a User object that we can access from everywhere within an app, but it needs to be optional, because some parts of the app don't have a logged in user, for example in the login screen.
Most parts of the app do require a logged in user but still need to work with the optional user, even if we know that those parts of the app require to have a logged in user and so the user should actually be non-optional.

The idea of "Locks and Keys" proposes to treat the required objects (the logged in user in this case) as keys to unlock those parts of the app.

As an example, I will use the screen PaymentView, which requires the logged in user.

```swift
struct PaymentView: View {
    @StateObject private var viewModel = PaymentViewModel()

    var body: some View {
        //...
    }
}

class PaymentViewModel: ObservableObject {
    var user: User? {
        return DataStore.shared.user
    }
}
```

I haven't applied the "Locks and Keys" principle yet and the view model is forced to access the logged in user as an optional.
Every time that the user is accessed, I need to make a tedious nil check and it's not clear how the case of nil should be handled.
Let's improve that.

```swift
struct PaymentView: View {
    @StateObject private var viewModel: PaymentViewModel

    init?() {
        guard let user = DataStore.shared.user else { return nil }
        self._viewModel = .init(wrappedValue: .init(user: user))
    }

    var body: some View {
        //...
    }
}

class PaymentViewModel: ObservableObject {
    let user: User

    init(user: User) {
        self.user = user
    }
}
```

In the view model, I changed the user property to be stored instead of computed and I'm assigning it a value in the initializer.
In the view, I provide a failable initializer (init with a question mark), which creates the viewmodel with a logged in, non-optional user.
If the user turns out to be nil, the construction of the view fails and evaluates to the equivalent of EmptyView().
The beauty of this approach is that all the places which use this view, don't need any changes.
I can still create them as usual. 

```swift
VStack {
    PaymentView()
}
```

Even though the initializer of the view is failable, it doesn't need to be checked for nil. If it fails, the view will be omitted. This was a very pleasant suprise for me :)

This is all that is needed for views which just require the user as static data, that doesn't change in the lifetime of this view.

However, this particular view requires to be updated when the user changes.
We can make a small addition to support this as well:

```swift
struct PaymentView: View {
    @StateObject private var viewModel: PaymentViewModel

    @ObservedObject private var dataStore = DataStore.shared

    init?() {
        guard let user = DataStore.shared.user else { return nil }
        self._viewModel = .init(wrappedValue: .init(user: user))
    }

    var body: some View {
        //...
        .onChange(of: dataStore.user) { user in
            if let user {
                viewModel.user = user
            } else {
                //optionally dismiss the view or navigate the user to the login screen.
            }
        }
    }
}

class PaymentViewModel: ObservableObject {
    @Published var user: User

    init(user: User) {
        self.user = user
    }
}
```

In the view model, I changed the user property from let to var, so that it's mutable and I added @Published to it.
In the view, I added an observer for the DataStore object, and in the body, I added a onChange modifier to listen to changes of the user in the dataStore object.
When the user changes, it is assigned to the user property in the view model.

That's it.
Now we have a view and a view model which can only exist with a logged in user, which is non-optional.
The user can change and the view will update automatically. 
If the user changes to nil, we can dismiss the view or navigate the app to the login screen, or whatever is the most appropriate action in this case, which is also a neat and useful result of this approach.

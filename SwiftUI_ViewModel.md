# ViewModels in SwiftUI

For SwiftUI arguably the most common architectural pattern is MVVM. It as simple as having a View which has a reference to an instance of an `Observable` ViewModel class.

This little article is a recommendation how to apply this pattern in a modern and elegant way.
It assumes a familiarity with SwiftUI and is not intended as a tutorial how to use ViewModels in SwiftUI.

## Naming and Nesting Types

Every ViewModel strictly belongs to a specific View. If we have a `ProductView`, a common naming convention would be to name the ViewModel `ProductViewModel`.
My recommendation however is to nest it inside of the `ProductView` and to just name it `ViewModel`:

```swift
struct ProductView: View {
    @State private var viewModel = ViewModel()
    
    var body: some View {
        Text("my product")
    }
}
```

```swift
extension ProductView {
    @Observable final class ViewModel {
        
    }
}
```

The ViewModel should be in a separate swift file and I recommend to name it the same as the View and to add "+ViewModel" to it:
In this example it would be `ProductView+ViewModel.swift`.

## ViewModel Parameters

If we have a ViewModel which needs to be passed parameters to initialize it, I don't recommend to make the ViewModel a parameter of the View initializer.
Instead, we should pass the parameters needed for the ViewModel to the View initializer and create a ViewModel there:

```swift
struct ProductView: View {
    @State private var viewModel: ViewModel
    
    init(productName: String) {
        viewModel = .init(productName: productName)
    }
    
    var body: some View {
        Text(viewModel.productName)
    }
}
```

```swift
extension ProductView {
    @Observable final class ViewModel {
        var productName: String
        
        init(productName: String) {
            self.productName = productName
        }
    }
}
```

With that approach, we don't need to know anything about the ViewModel when we are creating Views and can completely hide and ignore the details of the ViewModel inside of the View that it belongs to.

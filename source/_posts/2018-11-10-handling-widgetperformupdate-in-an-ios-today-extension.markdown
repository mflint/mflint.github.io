---
layout: post
title: "Handling widgetPerformUpdate in an iOS Today extension"
date: 2018-11-10 12:59:02 +0000
comments: true
categories: [swift,best practices]
---
Apple's documentation about the `NCWidgetProviding` protocol, and `widgetPerformUpdate:` function in particular are rather sparse, and most posts on StackOverflow seem to have a very simplistic view about how this callback should be coded. So here are my tips.

<!-- more -->

Most examples you'll see for `widgetPerformUpdate:` look rather like this:

    func widgetPerformUpdate(completionHandler: (@escaping (NCUpdateResult) -> Void)) {
        let resultModel = performExpensiveNetworkOperation()
        label.text = resultModel.someValue
        completionHandler(NCUpdateResult.newData)
    }

... which would probably be OK, most of the time, but there are very few examples of expensive work being done asynchronously.

There are, however, three _big_ hints that this is the right thing to do:

1. Apple's documentation says _"It's expected that the widget will perform the work to update asynchronously and off the main thread as much as possible"_. (The `widgetPerformUpdate:` call does indeed arrive on the main thread)
2. the `completionHandler` block is annotated `@escaping`, implying that it will outlive the scope of the `widgetPerformUpdate:` function itself
3. the function returns `Void`; if this were intended to be a _blocking_ call, then the func would simply have an `NCUpdateResult` return value instead of providing a completion block

### Doing it right

The correct way to do this depends on whether your long-running operation is synchronous (blocking), or asychronous with a completion block, or asynchronous with a delegate. Here are examples of all three.


#### Performing a synchronous (blocking) operation

    class SynchronousCallExampleViewController: UIViewController, NCWidgetProviding {
        private let dataSource = DataSource()
        private var data: Data?

        private func updateUI() {
            DispatchQueue.main.async {
                // update UI components
            }
        }

        func widgetPerformUpdate(completionHandler: @escaping (NCUpdateResult) -> Void) {
            DispatchQueue.global().async {
                // fetch data on a background thread
                self.data = self.dataSource.fetchData()
                self.updateUI()
                completionHandler(.newData)
            }
        }
    }

#### Performing an asynchronous operation with a completion block

    class AsynchronousCallWithCompletionBlockExampleViewController: UIViewController, NCWidgetProviding {
        private let dataSource = DataSource()
        private var data: Data?

        private func updateUI() {
            DispatchQueue.main.async {
                // update UI components
            }
        }

        func widgetPerformUpdate(completionHandler: @escaping (NCUpdateResult) -> Void) {
            DispatchQueue.global().async {
                // fetch data on a background thread
                self.dataSource.fetchDataAsync({ [weak self] (data) in
                    guard let self = self else { return }

                    self.data = data
                    self.updateUI()
                    completionHandler(.newData)
                })
            }
        }
    }


#### Performing an asynchronous operation with a delegate callback

    class AsynchronousCallWithDelegateExampleViewController: UIViewController, NCWidgetProviding, DataSourceDelegate {
        private var dataSource = DataSource()
        private var widgetCompletionHandler: ((NCUpdateResult) -> Void)?
        private var data: Data?
        
        private func updateUI() {
            DispatchQueue.main.async {
                // update UI components
            }
        }
        
        func widgetPerformUpdate(completionHandler: @escaping (NCUpdateResult) -> Void) {
            widgetCompletionHandler = completionHandler
            
            dataSource.delegate = self
            DispatchQueue.global().async {
                // fetch data on a background thread
                self.dataSource.fetchDataAsync()
            }
        }
        
        func dataDidUpdate(dataSource: DataSource) {
            data = dataSource.data
            updateUI()
            
            if let completionHandler = widgetCompletionHandler {
                completionHandler(.newData)
                widgetCompletionHandler = nil
            }
        }
    }


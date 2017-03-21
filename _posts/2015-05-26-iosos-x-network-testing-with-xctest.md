---
ID: 118
post_title: iOS/OS X network testing with XCTest
author: Lorenzo Bulfone
post_date: 2015-05-26 17:47:29
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/05/26/iosos-x-network-testing-with-xctest/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/119951034014/iosos-x-network-testing-with-xctest
tumblr_mikamayhem_id:
  - "119951034014"
---
<p>I recently worked on a OS X app which had to keep in sync with a database running on a remote server, to interact with a hardware device via serial port and to allow the user to edit said database with information coming from the hardware device. Since reinventing the wheel is never a good idea, I decided to use various frameworks for those tasks: <a href="https://github.com/armadsen/ORSSerialPort">ORSSerialPort</a> to handle the serial communication, <a href="https://github.com/AFNetworking/AFNetworking">AFNetworking</a> for the networking, Apple’s own <a href="https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CoreData/Articles/cdTechnologyOverview.html#//apple_ref/doc/uid/TP40009296-SW1">Core Data</a> for the local database and XCTest together with <a href="https://github.com/AliSoftware/OHHTTPStubs">OHHTTPStubs</a> for the testing. The networking part was by far the easiest to develop but also the hardest to test, due to its asynchronicity.</p>

<p>This is basically how XCTest works: you create subclasses of <code>XCTestCase</code>, and then define instance methods with their name starting with “test”. Inside these methods you can check for whatever condition you need to test. Here&rsquo;s a very simple example:</p>

<pre><code>- (void)testReverseString{ 

    NSString *originalString = @"test";
    NSString *reversedString = [self.controllerToTest reverseString:originalString];
    NSString *expectedReversedString = @"tset";

    XCTAssertEqualObjects(expectedReversedString, reversedString);
}
</code></pre>

<p><strong>XCTAssert</strong> allows to easily compare variables and to let tests fail/succeed based on the result, but it is not enough when it comes to testing the networking code. For example, I needed to test a GET request that fetched the user database from the remote server and tried to do so this way:</p>

<pre><code>- (void)testSuccessfulUserFetch
{

    // Perform fetch
    [self.networkManager fetchUsers];

    // Test if the number of users created in the local database is equal to the one in the remote database
    NSArray *localUserDatabase = [User usersInManagedObjectContext:self.moc];
    XCTAssertEqual([localUserDatabase count], 10);

}
</code></pre>

<p>The test failed even if the GET was performed with success, because the local database was tested right after the network request was started and not after its completion.</p>

<p>Luckily Xcode 6 introduced a new feature in XCTest that allows to test asynchronous code as well: <strong>XCTestExpectation</strong>. It’s as easy as declaring an expectation and setting a timeout for that expectation. The only other thing that’s needed is of course to check for whatever condition has to be met and to fulfill the expectation if that happens.</p>

<p>The obvious place to check and fulfill my network expectations was inside the success/failure blocks that my network manager used, but since that code was written way before I decided to write tests for it, there was no easy way to do so without refactoring. At first I felt like it was not a very good idea to refactor the network manager just to be able to test it, but I’m glad I did: while the existing code was working fine, refactoring it and being able to test it afterwards helped a lot while further developing it.</p>

<p>This is how I changed on of my NetworkManger.m methods to use optional success/failure blocks:</p>

<pre><code>typedef void(^SuccessBlock)(AFHTTPRequestOperation *operation, id responseObject);
typedef void(^FailureBlock)(AFHTTPRequestOperation *operation, NSError *error);

// Perform network request for user database
- (void)fetchUsersWithSuccess:(SuccessBlock)optionalSuccessBlock
         withFailure:(FailureBlock)optionalFailureBlock
{
    // Create success block
    SuccessBlock successBlock = ^(AFHTTPRequestOperation *operation, id responseObject){   

        // Load users into the database
        NSArray *usersArray = (NSArray *)responseObject;
        [User loadUsersFromArray:usersArray inManagedObjectContext:self.managedObjectContext];

        // Optional success block, if given
        if (optionalSuccessBlock) optionalSuccessBlock(operation, responseObject);
    };  

    // Create failure block
    FailureBlock failureBlock = ^(AFHTTPRequestOperation *operation, NSError *error){

        // Optional failure block, if given
        if (optionalFailureBlock) optionalFailureBlock(operation, error);
    };  

    // Perform request to server
    [self.manager GET:self.userSyncURL parameters:nil success:successBlock failure:failureBlock];
}
</code></pre>

<p>And here’s what the test method looks like:</p>

<pre><code>- (void)testSuccessfulUserFetch{
    // Create expectation
    XCTestExpectation *countExpectation = [self expectationWithDescription:@"User count expectation"];

    // Create custom success block
    SuccessBlock successBlock = ^(AFHTTPRequestOperation *operation, id responseObject){
        // Check and fulfill expectation
        [self checkUsersAndFullfillCountExpectation:countExpectation];
    };

  // Wait for expectations to be fulfilled. If timeout is met, test fails.
  [self waitForExpectationsWithTimeout:5.0 handler:^(NSError *error) {
    if (error) {
      NSLog(@"Timeout Error: %@", error);
    }
  }];
}

// Check expectations for successful user fetch
- (void)checkUsersAndFullfillExpectation:(XCTestExpectation *)countExpectation{

  // Check the users count in the database
  NSArray *users = [User usersInManagedObjectContext:self.moc];
  if ([users count] == 10){
    [countExpectation fulfill];
  }
}
</code></pre>
Let's summarize the best practices of using the microservice architecture model for applications, which are the following:
* Design for failure: In a microservice architecture, there are many interactions between the components, most of which are happening via remote calls and events. This increases the chance of various failures, including network timeouts, client errors, and many more. Build the system thinking of every possible failure scenario and different ways to proceed with it.
* Embrace automation: Having more independent components requires much stricter checks in order to achieve stable integration between the services. Investing in solid automation is absolutely necessary in order to achieve a high degree of reliability and ensure all changes are safe to be deployed.
* Don't ship hierarchy: Instead of using a service-per-team model, try to define the clear domains and business capabilities around which the code is structured and see how the components interact with each other. It is not easy to achieve perfect composition, but you will be highly rewarded for it.
* Invest in integration testing.
* Keep backward compatibility in mind: Always remember to keep your changes backward compatible to ensure that new changes are safe to deploy. Additionally, use techniques such as versioning, which we are going to cover in the next chapter of the book.


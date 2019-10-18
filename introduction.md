## Introduction to Continuous strategies

### Why we use that?

You have a project. You have a team working on this project. Your team deliver some amount of features/per day/per week/ per sprint. All participants of your team want to be sure that your repository contains stable and covered with specs code.

The needs associated with merging code grows every day and you also want some git branch after code base updating being sent to some stage(dev/staging/production etc.)

### Under control!

All of these cases could be handled with continuous strategies!

- Continuous integration
- Continuous delivery
- Continuous deployment

### Continuous integration

It's is a development practice that requires developers to integrate code into a shared repository several times a day.

Each check-in is then verified by an automated build, allowing teams to detect problems early.

![ci](images/ci.jpg)

### Benefits

- Say goodbye to long and tense integrations
- Increase visibility enabling greater communication
- Catch issues early and nip them in the bud
- Spend less time debugging and more time adding features
- Build a solid foundation
- Stop waiting to find out if your code’s going to work
- Reduce integration problems allowing you to deliver software more rapidly

### Continuous delivery

Continuous delivery is an extension of continuous integration to make sure that you can release new changes to your customers quickly in a sustainable way. This means that on top of having automated your testing, you also have automated your release process and you can deploy your application at any point of time by clicking on a button.

With continuous delivery, you can decide to release daily, weekly, fortnightly, or whatever suits your business requirements.

### Benefits

- The complexity of deploying software has been taken away. Your team doesn't have to spend days preparing for a release anymore.
- You can release more often, thus accelerating the feedback loop with your customers.
- There is much less pressure on decisions for small changes, hence encouraging iterating faster.

### Continuous deployment

Continuous deployment goes one step further than continuous delivery. With this practice, every change that passes all stages of your production pipeline is released to your customers. There's no human intervention, and only a failed test will prevent a new change to be deployed to production.

It is an excellent way to accelerate the feedback loop with your customers and take pressure off the team as there isn't a **Release Day** anymore. Developers can focus on building software, and they see their work go live minutes after they've finished working on it.

### Benefits

- You can develop faster as there's no need to pause development for releases. Deployments pipelines are triggered automatically for every change.
- Releases are less risky and easier to fix in case of problem as you deploy small batches of changes.
- Customers see a continuous stream of improvements, and quality increases every day, instead of every month, quarter or year.

![ci](images/cd_vs_cd.jpg)

### Final picture

![ci](images/ci_cd_cd.png)



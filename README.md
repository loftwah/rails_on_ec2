# Rails on EC2 with GitHub Actions

**Step 1: Create a new Ruby on Rails project**

1. Install Ruby on Rails on your local system if you haven't already.
2. Run the command `rails new gday_mate` in your terminal to create a new Rails application named "gday\_mate!".
3. Navigate to the `gday_mate` directory and run `rails generate controller Welcome index` to create a new controller named "Welcome" with an action called "index".
4. Open the `config/routes.rb` file and set the root route by adding `root 'welcome#index'`.
5. Run `rails server` in your terminal, and your "Gday Mate!" application should be up and running at `localhost:3000` on your local machine.

**Step 2: Push the project to a GitHub repository**

1. Initialize a Git repository in your Rails project by running `git init`.
2. Add all files to the Git repository with `git add .`.
3. Commit the changes with `git commit -m "Initial commit"`.
4. Create a new repository on your GitHub account. Let's call it `rails_gday_mate`.
5. Link your local Git repository to your GitHub repository with `git remote add origin https://github.com/<your-username>/rails_gday_mate.git`.
6. Push your local changes to the GitHub repository with `git push -u origin main`.

**Step 3: Set up a GitHub Actions workflow to automatically deploy the app to AWS EC2 when changes are made to the main branch**

0. Create an AWS EC2 micro instance in the `ap-southeast-2` region. Make sure Ruby, Rails, and other necessary software are installed. Create an SSH key for the instance and save the private key. You will use it in the GitHub Secrets.

1. In your GitHub repository settings, add the following secrets:

   * `AWS_ACCESS_KEY_ID`: Your AWS access key
   * `AWS_SECRET_ACCESS_KEY`: Your AWS secret access key
   * `AWS_SSH_PRIVATE_KEY`: The private key for the SSH key pair created for your EC2 instance
   * `AWS_HOST`: The public IP address of your EC2 instance
   * `AWS_USERNAME`: The username for your EC2 instance (usually `ubuntu` or `ec2-user`)

2. Create a new file in the `.github/workflows` directory, perhaps named `deploy.yml`.

3. In this file, define a workflow that triggers whenever changes are pushed to the main branch. Here is an updated example that takes your requirements into account:

```yaml
name: Deploy to AWS EC2

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Ruby
      uses: ruby/setup-ruby@v1
      with:
        ruby-version: 3.0

    - name: Install dependencies
      run: bundle install

    - name: Copy files via scp
      uses: appleboy/scp-action@master
      with:
        host: ${{ secrets.AWS_HOST }}
        username: ${{ secrets.AWS_USERNAME }}
        key: ${{ secrets.AWS_SSH_PRIVATE_KEY }}
        source: "."
        target: "/home/${{ secrets.AWS_USERNAME }}/gday_mate"

    - name: Execute remote ssh commands
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.AWS_HOST }}
        username: ${{ secrets.AWS_USERNAME }}
        key: ${{ secrets.AWS_SSH_PRIVATE_KEY }}
        script: |
          cd /home/${{ secrets.AWS_USERNAME }}/gday_mate
          bundle install
          rails db:migrate RAILS_ENV=production
          rails server -b 0.0.0.0 -d -e production
```

This workflow first sets up the Ruby environment and installs the dependencies on the GitHub Actions runner. Then, it copies the Rails application to the EC2 instance and installs the dependencies again on the server. Finally, it starts the Rails server in production mode, binding it to all IP addresses (`-b 0.0.0.0`) and running it as a daemon (`-d`).

Now, when you push changes to the main branch, your Rails "Gday Mate!" application should be deployed to the specified EC2 instance. You can show the audience the app by accessing the public IP address of the instance on the default Rails port (3000) in the browser. For example: `http://<public-ip-address>:3000`.

## Talk

### Minute 1: Introduction and Setup

Briefly introduce yourself and the topic of the talk.
Describe the overall goal: Leveraging open source tools to create a 'Gday Mate!' app in Ruby on Rails, push it to GitHub, and deploy it to AWS EC2 via GitHub Actions.

### Minute 2: Creating the Rails App

Quickly go over the steps of creating a new Ruby on Rails project, generating a controller, setting the root route, and running the app locally.

### Minute 3: Pushing the Project to GitHub

Explain how to initialize a Git repository, add and commit changes, and push to a GitHub repository.

### Minutes 4-5: Deploying the App to AWS EC2 with GitHub Actions

Spend a minute explaining what GitHub Actions is and its benefits.
Talk about setting up the EC2 instance and adding the necessary GitHub Secrets.
Quickly walk through the main parts of the GitHub Actions workflow file and what each step does.
Finally, emphasize that whenever changes are made to the main branch, the app will be automatically deployed to the EC2 instance.

### Closing Moments

Show the live app running on the EC2 instance by accessing it via the public IP in the browser.
Wrap up the talk by highlighting the power of open source tools for efficient software development and deployment, and encourage attendees to explore more.

> Note: Remember, you'll be moving quickly through the steps. Be sure to speak clearly and concisely, and consider having visuals or slides to help illustrate the points you're making.

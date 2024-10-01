# Python-Email-Notifier

Repo Activity Emailers Using Python.

Below is a Python script that automates sending daily email reports about your GitHub repository's activities. This script will use the smtplib library to send emails and the requests library to fetch GitHub data.

Requirements.

1. Install the necessary libraries:

pip install requests

2. Make sure to enable "Less secure app access" if you're using Gmail (or consider using an App Password with 2FA).

Python Script:

import smtplib
import requests
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from datetime import datetime, timedelta

# Configuration
GITHUB_USERNAME = 'your_github_username'
GITHUB_REPO = 'your_github_repo'
EMAIL_SENDER = 'your_email@example.com'
EMAIL_RECEIVER = 'recipient_email@example.com'
EMAIL_SUBJECT = f'Daily Report for {GITHUB_REPO}'
SMTP_SERVER = 'smtp.gmail.com'
SMTP_PORT = 587
EMAIL_PASSWORD = 'your_email_password'  # Use App Password if 2FA is enabled

# Function to fetch GitHub repository activities
def fetch_github_activities():
    end_date = datetime.now()
    start_date = end_date - timedelta(days=1)
    url = f'https://api.github.com/repos/{GITHUB_USERNAME}/{GITHUB_REPO}/events'
    response = requests.get(url)
    
    if response.status_code != 200:
        return f"Error fetching data: {response.status_code}"

    events = response.json()
    report = []

    for event in events:
        created_at = datetime.strptime(event['created_at'], '%Y-%m-%dT%H:%M:%SZ')
        if start_date <= created_at <= end_date:
            report.append(f"{event['type']} by {event['actor']['login']} at {event['created_at']}")

    return "\n".join(report) if report else "No activities found."

# Function to send email
def send_email(report):
    msg = MIMEMultipart()
    msg['From'] = EMAIL_SENDER
    msg['To'] = EMAIL_RECEIVER
    msg['Subject'] = EMAIL_SUBJECT

    body = f"Daily activities report for {GITHUB_REPO}:\n\n{report}"
    msg.attach(MIMEText(body, 'plain'))

    try:
        with smtplib.SMTP(SMTP_SERVER, SMTP_PORT) as server:
            server.starttls()
            server.login(EMAIL_SENDER, EMAIL_PASSWORD)
            server.send_message(msg)
        print("Email sent successfully.")
    except Exception as e:
        print(f"Error sending email: {e}")

# Main function
def main():
    report = fetch_github_activities()
    send_email(report)

if __name__ == "__main__":
    main()

Instructions:

1. Replace the placeholders in the configuration section (GITHUB_USERNAME, GITHUB_REPO, EMAIL_SENDER, EMAIL_RECEIVER, and EMAIL_PASSWORD) with your actual values.

2. Schedule the script to run daily:

On Windows, you can use Task Scheduler.

On Unix-based systems, you can use cron.

Example Cron Job.

To schedule the script to run daily at 9 AM, add the following line to your crontab (crontab -e):

0 9 * * * /usr/bin/python3 /path/to/your/script.py

Note:

Make sure to handle any sensitive information carefully (like email passwords).

Adjust the error handling and logging as needed for your use case.

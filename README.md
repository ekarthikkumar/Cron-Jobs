Cron-Jobs
=========

**Create python file and stored in location**


`For Eg : /home/karthik/cronjobs/task.py`

`task.py`

	#!/usr/bin/python
	import MySQLdb
	import sys
	import smtplib
	from email.MIMEMultipart import MIMEMultipart
	from email.MIMEText import MIMEText
	from email.MIMEImage import MIMEImage

	db = MySQLdb.connect("xxx.xxx.xxx.xxx","username","password","DB")
	cursor = db.cursor()
	sql = "select * from users"
	try:
		# Execute the SQL command
		cursor.execute(sql)
		# Fetch all the rows in a list of lists.
		results = cursor.fetchall()
		for row in results:
			if not row[6]:
				email = row[5]
				
				# Create the root message and fill in the from, to, and subject headers
				msgRoot = MIMEMultipart('related')
				msgRoot['Subject'] = 'Subjects'
				msgRoot['From'] = 'from@domain.com'
				msgRoot['To'] = email
				msgRoot.preamble = 'This is a multi-part message in MIME format.'

				# Encapsulate the plain and HTML versions of the message body in an
				# 'alternative' part, so message agents can decide which they want to display.
				msgAlternative = MIMEMultipart('alternative')
				msgRoot.attach(msgAlternative)

				msgText = MIMEText('This is the alternative plain text message.')
				msgAlternative.attach(msgText)

				# We reference the image in the IMG SRC attribute by the ID we give it below
				msgText = MIMEText("<!DOCTYPE html><html lang='en'><head></head><body style='font-size:16px;  font-family:Calibri;'><br><div style='margin-left:50px;'><p> Hi <b>%s</b>, <br/><br/> We are excited that you are now part of the ReReZe peer-to-peer rental marketplace. <br/><br/>Please click <a href='http://domainname/verify_user?token_id=%s'> here </a> to activate your account.<br/></p><br><p>The ReReZe Team</p><img src= 'cid:image1'  alt='ReReZe logo image' style='width: 130px; float:left;'></div></body></html>" %(row[5],row[10]),"HTML")
				
				msgAlternative.attach(msgText)

				# This example assumes the image is in the current directory
				fp = open('image/path/img.png', 'rb')
				msgImage = MIMEImage(fp.read())
				fp.close()

				# Define the image's ID as referenced above
				msgImage.add_header('Content-ID', '<image1>')
				msgRoot.attach(msgImage)

				#User credentials for mail configuration
				username = 'from@domain.com'  
				password = 'password'

				# Send the email (this example assumes SMTP authentication is required)

				server = smtplib.SMTP('smtp.gmail.com')  
				server.starttls()  
				server.login(username,password)  
				server.sendmail('from@domain.com', email, msgRoot.as_string())  
				server.quit()
				update_sql = "update users set is_sent=1 where id=%d" %row[0]
				cursor.execute(update_sql)
				db.commit()
	except Exception as e:print e
	db.close()

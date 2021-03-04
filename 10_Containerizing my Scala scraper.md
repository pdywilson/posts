# Containerizing my Scala scraper

I'm really keen on this driver update for my UR22 soundcard from Steinberg, so I decided to write a program that notifies me via email if the driver website gets updated. For this task I decided to write a small Scala script, containerize it with Docker and running it on a Google Cloud VM instance with help of the scheduling tool cron.

To write up the script I initially used [Scastie](https://scastie.scala-lang.org/), later I moved to using sbt and scala locally as well as on the VM instance. With the help of the Scalascraper and Regex libraries, it was straight forward to write a scraper that pulls the most recent update-date from the driver website.

```
import scala.util.matching.Regex
import net.ruippeixotog.scalascraper.browser.{HtmlUnitBrowser, JsoupBrowser}
import net.ruippeixotog.scalascraper.dsl.DSL.Extract._
import net.ruippeixotog.scalascraper.dsl.DSL._

object Scalaguy {
  
  def main(args: Array[String]): Unit = {
    
    val numberPattern: Regex = "[A-Z][a-z]* [0-9]+, 2021".r

    val browser = JsoupBrowser()
    val doc2 = browser.get("https://www.steinberg.net/en/support/downloads_hardware/yamaha_steinberg_usb_driver.html")
    val date = numberPattern.findAllMatchIn(doc2 >> allText).toList

    val outp = date(0).toString

    if (outp == "January 12, 2021") {
      println("No updates yet.")
    }

    else {
      println("Website has been updated! Check it out!")
	//TODO: send email
    }
  }
}
```


What's left to do is to set up the smtp with my gmail (watch out with using the password freely here, I used a throwaway e-mail here).


import courier._, Defaults._
val mailer = Mailer("smtp.gmail.com", 587)
                   .auth(true)
                   .as("myemail@gmail.com", "mypassword")
                   .startTls(true)()
      mailer(Envelope.from("me" `@` "gmail.com")
            .to("me" `@` "gmail.com")
            .subject("UR22 Update")
            .content(Text(outp)))

Simple as that! ... You might think. This works on Scastie, but it took me a while to get it to work on the Server. After a lot of trying around I figured out that the JVM instance gets killed too early and the email never makes it out! Therefore I added a 'Thread.sleep(5000)' at the end, and this made it work!!

Finally, to containerize the app, we simply need to run a few convenient sbt commands. I found this great guide on how to do it: https://www.freecodecamp.org/news/how-to-dockerise-a-scala-and-akka-http-application-the-easy-way-23310fc880fa/.

Basically we need to add 
```
addSbtPlugin("com.typesafe.sbt" % "sbt-native-packager" % "1.3.6")
```
to project/plugin.sbt, and 
```
enablePlugins(JavaAppPackaging)
```
to build.sbt. This enables us to use 'sbt stage' and run our app. If we have docker installed, we can then run sbt docker:stage to stage it and sbt docker:publishLocal to create the image. Run the image as usual per docker run <image>. 

Our good old friend cron can be used to run the program every few hours. Using crontab -e lets us input the schedule time and the command to run. In this case it is good practice used docker run --rm <image>  to run my app, so that it automatically removes the container when it's done.



Other notes:
- How to send emails via command-line: https://sylvaindurand.org/send-emails-with-msmtp/
- Some commands for docker clean up:
```
docker container stop $(docker container ls –aq)

docker container rm $(docker container ls –aq)
```

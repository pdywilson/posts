```
import net.ruippeixotog.scalascraper.browser.JsoupBrowser
import net.ruippeixotog.scalascraper.dsl.DSL.Extract._
import net.ruippeixotog.scalascraper.dsl.DSL._
import scala.collection.parallel._
import scala.concurrent.{Await, Future}
import scala.concurrent.duration._
import scala.concurrent.ExecutionContext.Implicits.global
import scala.collection.parallel.CollectionConverters._
import scala.concurrent._
import java.util.concurrent.Executors
import scala.util.matching.Regex




object Main {
  def main(args: Array[String]): Unit = {
    val t0 = System.currentTimeMillis()
    
    /** 
    * Part 1
    */
    // val browser = JsoupBrowser()    

    // val url = "https://en.wikipedia.org/wiki/Scala_(programming_language)"

    // val doc = browser.get(url)

    // val numberPattern = "[0-9]{1,3}".r

    // val rentList: List[Int] = 
    //   (doc >> elementList("div"))
    //     .map(_ >> allText)
    //     .map(elt => numberPattern.findAllMatchIn(elt))
    //     .flatten
    //     .map(d => d.toString.toInt)

    // println("The Average is: "+rentList.sum/rentList.length)

    /** 
    * End Part 1
    */




    /** 
    * Part 2
    */
    // val url = "https://en.wikipedia.org/wiki/Scala_(programming_language)"
      
    // def processOneUrl(url: String): List[Int] = { 
    //   val numberPattern: Regex = "[0-9]{1,3}".r
    //   val browser = JsoupBrowser()
    //   val doc = browser.get(url)
    //   val rentList = (doc >> elementList("div"))
    //                 .map(_ >> allText)
    //                 .map(elt => numberPattern.findAllMatchIn(elt))
    //                 .flatten
    //                 .map(elt => elt.toString.toInt)
    //   rentList
    //  }


    // val scrape_result = (1 to 100).par.map(x => processOneUrl(url))

    // val rentList = scrape_result.map(x => x.sum/x.length)

    // println(rentList)

    // println("Elapsed time: "+(System.currentTimeMillis()-t0)+"ms")

    /** 
    * End Part 2
    */



    /** 
    * Part 3
    */
    val url = "https://en.wikipedia.org/wiki/Scala_(programming_language)"

    def mean(l: Seq[Int]) = {
      if(l.length == 0) 0
      else l.sum/l.length
    }
      
    def processOneUrl(url: String): Future[List[Int]] = Future { 
      val numberPattern: Regex = "[0-9]{1,3}".r
      val browser = JsoupBrowser()
      val doc = browser.get(url)
      val rentList = 
        (doc >> elementList("div"))
          .map(_ >> allText)
          .map(elt => numberPattern.findAllMatchIn(elt))
          .flatten
          .map(elt => elt.toString.toInt)
      rentList
     } recover {case _ => List[Int]()}

    def scrape() = {
      val rentList = 
        (1 to 100).par
          .map(x => processOneUrl(url))
          .map(x => Await.result(x, 30.seconds))
          .map(x => mean(x))
      println(rentList)
    }

    var times = List[Int]()
    for (i <- 0 until 5) {
      val t0 = System.currentTimeMillis()
      scrape()
      val t1 = System.currentTimeMillis()
      times = times :+ (t1-t0).toInt
    }

    println("Times: "+times)
    println("Minimum time: "+times.min+"ms")

    /** 
    * Part 3
    */


  }
}

```
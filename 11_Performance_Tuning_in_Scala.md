import net.ruippeixotog.scalascraper.browser.JsoupBrowser
import scala.collection.parallel._
import scala.concurrent.{Await, Future}
import scala.concurrent.duration._
import scala.concurrent.ExecutionContext.Implicits.global
import scala.collection.parallel.CollectionConverters._
import scala.concurrent._
import java.util.concurrent.Executors



object HelloWorld {
  def main(args: Array[String]): Unit = {
    //implicit val ec = ExecutionContext.fromExecutor(new java.util.concurrent.ForkJoinPool(12))
    //implicit val ec = ExecutionContext.fromExecutor(Executors.newFixedThreadPool(6))
    
    println(Runtime.getRuntime().availableProcessors())
    
    val t0 = System.currentTimeMillis()
    
    val browser = JsoupBrowser()    
    
    val parvec = (0 to 50).toVector
    val mapper = parvec.map(_ => Future{browser.get("https://www.daft.ie/").toString.length} andThen {case x => println(x)})
    val reducer = mapper.map(x => Await.result(x, 20.seconds))
    
    println("elapsed time: "+(System.currentTimeMillis()-t0)/1000)

    
  }
}

// for daft.ie 1 sec, for daft.ie/property-for-rent/dublin?numBeds_from=2&numBeds_to=2 7 sec 
// counts number of characters on webpage: 116k for daft.ie, 1.6 Mio for daft.ie/property-for-rent/dublin?numBeds_from=2&numBeds_to=2





# 二分法

```java
import java.math.BigDecimal;
public class SquareRoot {
	//精确小数位数，越精确，效率越低
    public static int SCALE = 5;
	
	/**精确格式 如:String.format("0.%05d",1) = 0.00001*/
	private static Double SCALE_DOUBLE = Double.valueOf(String.format("0.%0"+SCALE+"d",1));
	
	public static BigDecimal sqrt(int num){
		
		if(num < 1){
			return BigDecimal.valueOf(-1);
		} else if(num == 1){
			return BigDecimal.ONE;
		}
	//首先定位到两个连续的整数范围内
		int max = num;
		int min = 0;
		//当两个整数相减=1时，说明已经定位到了整数范围如5-6之间
		while( (max - min) != 1){
			//使用二分查找法
			int mid = (max+min)/2;
			//计算中间数的乘积
			int j = mid*mid;
			if(j>num){
				max = mid;
			} else if(j < num){
				min = mid;
			} else {
				//直接相等,如3*3=9
				return BigDecimal.valueOf(mid);
			}
			  		
		}
		
		//System.out.println(String.format("整数区间(%d,%d)",min,max));
		//继续精确到指定的小数位
		return sqrt(num,BigDecimal.valueOf(min),BigDecimal.valueOf(max));
	}
	
	private static BigDecimal sqrt(int num,BigDecimal min,BigDecimal max){
		
		BigDecimal numb = BigDecimal.valueOf(num);
		BigDecimal mid = null;
		BigDecimal j = null;
		while(true){
			mid = min.add(max).divide(BigDecimal.valueOf(2));
			j = mid.multiply(mid);
			
			if(j.compareTo(numb) > 0){
				max = mid;
			} else {
				min = mid;
			}
			
            // 当差值在0.00001时说明精确到了小数点后5位
			if(numb.compareTo(j)>0 && 
			   numb.subtract(j).doubleValue() < SCALE_DOUBLE){
				break;
			}
			
		}
		return mid.setScale(SCALE,BigDecimal.ROUND_FLOOR);
 
	}
	
	
	public static void main(String []args) {
				     
			   System.out.println(sqrt(100));
			   System.out.println(sqrt(99));
			   System.out.println(sqrt(8));
 
 
    }
}
```



# 牛顿迭代法

```java
public class niudundiedai {
    public static double sqt(double c){
        if(c<0){
            return Double.NaN;
        }
        double err=1e-15;
        double t=c;//t代表参数c的平方根,先随便给t一个初始值c
        int count=0;//count没任何用,就是用来看计算过程的
        while(Math.abs(t-c/t)>err*t){//由于只是无限接近真正的平方根,所以比较c的估算平方根t,t的平方和初始c的差距,要小于指定误差,并计算次数

            t=(c/t+t)/2.0;//x2=(x1+c/x1)/2,不断计算x值,接近真是的值
            //count++;//计算的次数
            //System.out.print("第"+count+"次计算");
            //System.out.println(t);//每次计算的值,越来越接近了!
        }
        return t;
    }
    public static void main(String[] args) {
        double sqt = sqt(102);
        System.out.println("符合误差的最终值");
        System.out.println(sqt);
    }
}

```


# Chapter 16 - Refactoring SerialDate

The author of SerialDate is David Gilbert. David is clearly an experienced and competent programmer. We are going to explore the SerialDate class. 


## Case Introduction

This is not an activity of nastiness or arrogance. This is nothing more and nothing less than a professional review.

It is only through critiques like these that we will learn. Doctors do it. Pilots do it. Lawyers do it. And we programmers need to learn how to do it too.

At times, java.util.Date can be too precise - it represents an instant in time, accurate to 1/1000th of a second (with the date itself depending on the time-zone). Sometimes we just want to represent a particular day (e.g. 21 January 2015) without concerning ourselves about the time of day, or the time-zone, or anything else. That's what SerialDate is for.

## Two files which we are going to use  1. ComparisionCompactor.java and 2. ComparisionCompactorTest.java
Although the unit test code has covered everything from the unit test perspective but we have scope to refine or refactor original ComparisionCompactor class

## The code before refactoring
```java

/******************** Uses Constants by inheriting the interface ****************/
public interface MonthConstants {
	/** Constant for January. */
	public static final int JANUARY = 1;
	/** Constant for February. */
	public static final int FEBRUARY = 2;
	/** Constant for March. */
	public static final int MARCH = 3;
	/** Constant for April. */
	public static final int APRIL = 4;
	/** Constant for May. */
	public static final int MAY = 5;
	/** Constant for June. */
	public static final int JUNE = 6;
	/** Constant for July. */
	public static final int JULY = 7;

	/** Constant for August. */
	public static final int AUGUST = 8;

	/** Constant for September. */
	public static final int SEPTEMBER = 9;

	/** Constant for October. */
	public static final int OCTOBER = 10;

	/** Constant for November. */
	public static final int NOVEMBER = 11;

	/** Constant for December. */
	public static final int DECEMBER = 12;
}

/*********************** Creates an instance of the SerialDate class *******************/
public DayDate getNearestDayOfWeek(final Day targetDay) {
	int offsetToThisWeeksTarget = targetDay.index - getDayOfWeek().index;
	int offsetToFutureTarget = (offsetToThisWeeksTarget + 7) % 7;
	int offsetToPreviousTarget = offsetToFutureTarget - 7;
	if (offsetToFutureTarget > 3)
	return plusDays(offsetToPreviousTarget);
	else
	return plusDays(offsetToFutureTarget);
}

/**
* Creates a new date by adding the specified number of days to the base
* date.
*
* @param days the number of days to add (can be negative).
* @param base the base date.
*
* @return a new date.
*/
public static SerialDate addDays(final int days, final SerialDate base) {
	final int serialDayNumber = base.toSerial() + days;
	return SerialDate.createInstance(serialDayNumber);
}

/*************************** SerialDate class ********************/
public abstract class SerialDate implements Comparable,
	Serializable,
	MonthConstants {

	/** For serialization. */
	private static final long serialVersionUID = -293716040467423637L;

	/** Date format symbols. */
	public static final DateFormatSymbols
	DATE_FORMAT_SYMBOLS = new SimpleDateFormat().getDateFormatSymbols();

	/** The serial number for 1 January 1900. */
	public static final int SERIAL_LOWER_BOUND = 2;

	/** The serial number for 31 December 9999. */
	public static final int SERIAL_UPPER_BOUND = 2958465;

	/** The lowest year value supported by this date format. */
	public static final int MINIMUM_YEAR_SUPPORTED = 1900;

	/** The highest year value supported by this date format. */
	public static final int MAXIMUM_YEAR_SUPPORTED = 9999;

	
```

## Refactoring Findings

**1. Increase the test coverage for the SerialDate class**

Existing test coverage is only about ~50%. With the refactoring and newly added tests, the test coverage has increased to ~92%.

**2. Name of the class should be more descriptive ?**

This class is named SerialDate because it is implemented using a “serial number,” which happens to be the number of days since December 30th, 1899. This may be a quibble, but the representation is more of a relative offset than a serial number. 
So this name particularly descriptive. A more descriptive term might be “ordinal.”

**3. SerialDate is an abstract class - Wrong level of abstraction**

The name SerialDate implies an implementation. This class is an abstract class. There is no need to imply anything at all about the implementation. Indeed, there is good reason to hide the implementation!

**4. Use Enum instead of constants**

It's always better to use Enum instead of inheriting the classes with constants.

**5. Define the ownership of the attributes with the respective class**
The SerialDate is an abstract class that provides no foreshadowing of implementation, then it should not inform us
about a minimum or maximum year. So moved these variables (MINIMUM_YEAR_SUPPORTED, and MAXIMUM_YEAR_SUPPORTED) to  SpreadsheetDate class.

**6. Base classes should not know about their derivatives**
To fix this, we should use the ABSTRACT FACTORY pattern and create a DayDateFactory. This factory will create the instances of DayDate (formerly SerialDate).


## The code after refactoring
```java
/********************** Created an Enum instead of constants *********************/
public abstract class DayDate implements Comparable, Serializable {
	public static enum Month {
	JANUARY(1),
	FEBRUARY(2),
	MARCH(3),
	APRIL(4),
	MAY(5),
	JUNE(6),
	JULY(7),
	AUGUST(8),
	SEPTEMBER(9),
	OCTOBER(10),
	NOVEMBER(11),
	DECEMBER(12);

	Month(int index) {
		this.index = index;
	}
}

/********************** Created factory to create the instances *********************/
public abstract class DayDateFactory {
	private static DayDateFactory factory = new SpreadsheetDateFactory();
	public static void setInstance(DayDateFactory factory) {
		DayDateFactory.factory = factory;
	}
	protected abstract DayDate _makeDate(int ordinal);
	protected abstract DayDate _makeDate(int day, DayDate.Month month, int year);
	protected abstract DayDate _makeDate(int day, int month, int year);
	protected abstract DayDate _makeDate(java.util.Date date);
	protected abstract int _getMinimumYear();
	protected abstract int _getMaximumYear();
	
	public static DayDate makeDate(int ordinal) {
		return factory._makeDate(ordinal);
	}
	
	public static DayDate makeDate(int day, DayDate.Month month, int year) {
		return factory._makeDate(day, month, year);
	}
	public static DayDate makeDate(int day, int month, int year) {
		return factory._makeDate(day, month, year);
	}
	public static DayDate makeDate(java.util.Date date) {
		return factory._makeDate(date);
	}
	public static int getMinimumYear() {
		return factory._getMinimumYear();
	}
	public static int getMaximumYear() {
		return factory._getMaximumYear();
	}
}

public DayDate plusDays(int days) {
	return DayDateFactory.makeDate(getOrdinalDay() + days);
}

/*************************** SpreadsheetDate class ********************/
public class SpreadsheetDate extends DayDate {
	public static final int EARLIEST_DATE_ORDINAL = 2; // 1/1/1900
	public static final int LATEST_DATE_ORDINAL = 2958465; // 12/31/9999
	public static final int MINIMUM_YEAR_SUPPORTED = 1900;
	public static final int MAXIMUM_YEAR_SUPPORTED = 9999;

	private int ordinalDay;
	private int day;
	private Month month;
	
```


## Conclusion

- With the revision, the current code looks bit cleaner than when we checked it out. It took a little time, but it was worth it. Test coverage was increased, some bugs were fixed, the code was clarified and shrunk. 
- The next person to look at this code will hopefully find it easier to deal with than we did. That person will also
probably be able to clean it up a bit more than we did.

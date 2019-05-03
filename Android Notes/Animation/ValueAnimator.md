# **ValueAnimator**
- Author：WeicongLee
- Data：2019.05.02

## **一、ValueAnimator是整个属性动画机制当中最核心的一个类**
- 除了ViewPropertyAnimator和ObjectAnimator，还有第三个选择：ValueAnimator
- 它是ObjectAnimato和ViewPropertyAnimator的父类，但并不常用，主要是太基础了
- 不能像ObjectAnimator那样指定目标对象，更不能自动调用目标对象的setter方法来更新目标属性的值
- 只能通过渐变的方式来改变一个独立的数据，至于在数据更新后要做什么，全由你决定
- 不管做什么都要你自己来写

## **二、三者关系**
- ### 三种Animator：ViewPropertyAnimator、ObjectAnimator、ValueAnimator，它们其实是一种递进的关系：从左到右依次变得更加难用，也更加灵活。但是三者性能是一样的，因为其余两个的内部实现都是ValueAnimator。
- ### 区别在于便捷性以及功能的灵活性，选用的时候只要遵守一个原则：尽量用简单的
- ### **能用View.animate()实现就不用ObjectAnimator，能用ObjectAnimator就不用ValueAnimator**

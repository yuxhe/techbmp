#@Import只能用在类上
@Import({UserService.class,xxx.class})
导入定义的类，这个类里面又去导入其他类的定义哈；
对应的import的bean都将加入到spring容器中，这些在容器中bean名称是该类的全类名 ，比如com.yc.类名

@Import的三种用法主要包括：

1、直接填class数组方式   直接导入第三方的bean
2、ImportSelector方式【重点】   导入定义
3、ImportBeanDefinitionRegistrar方式   导入定义





#1）实现 ImportSelector接口
public class Myclass implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        return new String[]{"com.yc.Test.TestDemo3"}; //此处可动态 标记出多个待放入容器的类
    }
}

#2）并标注上使用ImportSelector方式

@Import({普通类TestDemo2.class,实现ImportSelector接口的类 Myclass.class})
public class TestDemo {
       .....
}

-----------------------------------------
打印容器中的类

/**
 * 打印容器中的组件测试
 */
public class AnnotationTestDemo {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext applicationContext=new AnnotationConfigApplicationContext(TestDemo.class);  //这里的参数代表要做操作的类

        String[] beanDefinitionNames = applicationContext.getBeanDefinitionNames();
        for (String name : beanDefinitionNames){
            System.out.println(name);
        }

    }
}

-------------------------

#ImportBeanDefinitionRegistrar方式 ，实现ImportBeanDefinitionRegistrar 接口

#1）自定义注册bean

public class Myclass2 implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata annotationMetadata, BeanDefinitionRegistry 
beanDefinitionRegistry) {

        //指定bean定义信息（包括bean的类型、作用域...）

        RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(TestDemo4.class);

        //注册一个bean指定bean名字（id）

        //这个名字可自定义指定
        beanDefinitionRegistry.registerBeanDefinition("TestDemo4444",rootBeanDefinition);

    }

}


#2）标注上使用ImportBeanDefinitionRegistrar方式

@Import({实现了ImportBeanDefinitionRegistrar的类 Myclass2.class})
public class TestDemo {

        @Bean
        public AccountDao2 accountDao222(){// 容器中的名字用的这个 哈 accountDao222
            return new AccountDao2();
        }

}


#总结

#第一种用法：@Import（{ 要导入的容器中的组件 } ）：容器会自动注册这个组件，id默认是全类名

 
#第二种用法：ImportSelector：返回需要导入的组件的全类名数组，springboot底层用的特别多【重点 】

 
#第三种用法：ImportBeanDefinitionRegistrar：手动注册bean到容器


#以上三种用法方式皆可混合在一个@Import中使用，特别注意第一种和第二种都是以全类名的方式注册，而第三中可自定义方式



修改容器中bean定义 1）修改实例化参数  2）修改属性的访问方式  3）可增加属性  4）实现bean动态代理等

#beanFactoryPostProcessor 修改bean定义,在实例化之前

  当你实现了这个接口的时候，可以对还没有初始化的bean的属性进行修改或添加

BeanFactoryPostProcessor 为spring在容器初始化时对外对外暴露的扩展点，Spring IoC容器允许BeanFactoryPostProcessor在容器加载注册BeanDefinition完成之后读取BeanDefinition(配置元数据)，并可以修改它

public interface BeanFactoryPostProcessor {
	void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;
}


1. beanFactoryPostProcessor接口可以在bean未被实例化之前获取bean的定义即配置元数据，然后根据需要进行更改。

2. beanFactoryPostProcessor里有方法  void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException; 此方法可以通过beanFactory可以获取bean的定义信息，并可以修改bean的定义信息。

3. 可以在spring配置文件中定义多个beanFactoryPostProcessor，执行顺序按照定义顺序执行，也可以根据order属性自己定义执行顺序


执行顺序 ：  BeanFactoryPostProcessor ---> 普通Bean构造方法 ---> 设置依赖或属性 ---> @PostConstruct ---> InitializingBean ---> initMethod


#FactoryBean 接口能产生实列及实例的类型 底层很多地方用到哈

FactoryBean:是一个Java Bean,但是它是一个能生产对象的工厂Bean




#BeanPostProcessor



#{} 预编译参数形式 ，${} 这个是替换方式哈


### 1. 자바 클래스 구조 



### 2. 클래스 읽어오기(ClassReader)

아래는 ClassPrinter의 예제코드이다

```java
package BCITest;

import org.objectweb.asm.*;

import static org.objectweb.asm.Opcodes.ASM6;

public class ClassPrinter extends ClassVisitor{

    public ClassPrinter() {
        super(ASM6);
    }

    @Override
    public void visit(int version, int access, String name, String signature, String superName, String[] interfaces) {
        System.out.println(name + " extends " + superName + " {");
    }

    @Override
    public void visitSource(String source, String debug) {

    }

    @Override
    public void visitOuterClass(String owner, String name, String desc) {

    }

    @Override
    public AnnotationVisitor visitAnnotation(String desc, boolean visible) {
        return null;
    }

    @Override
    public void visitAttribute(Attribute attr) {

    }

    @Override
    public FieldVisitor visitField(int access, String name, String desc, String signature, Object value) {
        System.out.println("    " + desc + " " + name);
        return null;
    }

    @Override
    public MethodVisitor visitMethod(int access, String name, String desc, String signature, String[] exceptions) {
        System.out.println("    " + name + desc);
        return null;
    }

    @Override
    public void visitEnd() {
        System.out.println("}");
    }
}
```

아래 코드는 위에서 정의한 클래스를 ClassReader를 이용해서 사용하는 예제이다

```java
package BCITest;

import org.objectweb.asm.ClassReader;
import java.io.IOException;

import java.lang.reflect.Method;

// TODO : 시스템 클래스 읽어오는거 나중에 추가하기;;;
public class AsmExampleMain {

    public static void main(String[] args) throws IOException{

        ClassPrinter cp = new ClassPrinter();
        ClassReader cr = new ClassReader("BCITest.HelloWorld");
        cr.accept(cp,0);
    }
}
```



### 3. 클래스 생성하기(ClassWriter)

```java
package BCITest;
public interface Comparable extends Mesurable {
	int LESS = -1;
	int EQUAL = 0;
	int GREATER = 1;
	int compareTo(Object o);
}
```



아래는 위와 같은 인터페이스를 ClassWriter를 이용해서 생성하는 예제코드이다

```java
package BCITest;

import org.objectweb.asm.ClassWriter;

import static org.objectweb.asm.Opcodes.*;

public class GeneratingClassExampleMain {

    public static class MyClassLoader extends ClassLoader {
        public Class defineClass(String name, byte[] b){
            return defineClass(name, b, 0, b.length);
        }
    }

    public static void main(String[] args) {
        ClassWriter cw = new ClassWriter(0);
        cw.visit(V1_8, ACC_PUBLIC + ACC_ABSTRACT + ACC_INTERFACE,
        "BCITest/Comparable", null, "java/lang/Object",
                new String[]{"BCITest/Measurable"});
        cw.visitField(ACC_PUBLIC + ACC_FINAL + ACC_STATIC, "LESS", "I",
                null, new Integer(-1)).visitEnd();

        cw.visitField(ACC_PUBLIC + ACC_FINAL + ACC_STATIC, "EQUAL", "I",
                null, new Integer(0)).visitEnd();

        cw.visitField(ACC_PUBLIC + ACC_FINAL + ACC_STATIC, "GREATER", "I",
                null, new Integer(1)).visitEnd();

        cw.visitMethod(ACC_PUBLIC + ACC_ABSTRACT, "compareTo", "(Ljava/lang/Object;)I", null, null).visitEnd();
        cw.visitEnd();

        Class c = new MyClassLoader().defineClass("BCITest.Comparable", cw.toByteArray());

        System.out.println(c.getName());
    }
}
```

### 4. 클래스 내용 변경하기(Transformer && Visitor)

VisitField가지고 메소드 만들면 에러납니다 ㅠ
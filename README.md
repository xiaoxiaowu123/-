# -
使用SamrtTable 表头分行记录
以下是我写的带图片的表头换行view 注：需要用回车符“\n”进行分行显示

主要的代码是重写measureHeight()和measureWidth()，draw();
measureHeight()由于是绘制所有的title高度，所以只能写死高度，我这用的是 * 4的处理，就最大行数
要通过getSplitString(String val)方法对有分行符的文本进行重描；
（主要参考smartTable下com.bin.david.form.data.format.draw.MultiLineDrawFormat）



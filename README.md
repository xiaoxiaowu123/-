# -
使用SamrtTable 表头分行记录
以下是我写的带图片的表头换行view 注：需要用回车符“\n”进行分行显示

主要的代码是重写measureHeight()和measureWidth()，draw();
measureHeight()由于是绘制所有的title高度，所以只能写死高度，我这用的是 * 4的处理，就最大行数
要通过getSplitString(String val)方法对有分行符的文本进行重描；
一下是完整代码（主要参考smartTable下com.bin.david.form.data.format.draw.MultiLineDrawFormat）

import android.graphics.Canvas;
import android.graphics.Paint;
import android.graphics.Rect;

import com.bin.david.form.core.TableConfig;
import com.bin.david.form.data.column.Column;
import com.bin.david.form.data.format.bg.ICellBackgroundFormat;
import com.bin.david.form.data.format.title.ImageResTitleDrawFormat;
import com.bin.david.form.data.format.title.TitleDrawFormat;
import com.bin.david.form.exception.TableException;
import com.bin.david.form.utils.DrawUtils;
import java.lang.ref.SoftReference;
import java.util.HashMap;
import java.util.Map;
public abstract class MultiLineTitleImageDrawFormat extends ImageResTitleDrawFormat {
private Map<String, SoftReference<String[]>> valueMap; //避免产生大量对象

public static final int LEFT = 0;
public static final int TOP = 1;
public static final int RIGHT = 2;
public static final int BOTTOM = 3;

private TitleDrawFormat textDrawFormat;
private int drawPadding;
private int direction;
private int verticalPadding;
private int horizontalPadding;
private Rect rect;

public MultiLineTitleImageDrawFormat(int imageWidth, int imageHeight, int drawPadding) {
    this(imageWidth, imageHeight, LEFT, drawPadding);
}

public MultiLineTitleImageDrawFormat(int imageWidth, int imageHeight, int direction, int drawPadding) {
    super(imageWidth, imageHeight);
    valueMap = new HashMap<>();
    textDrawFormat = new TitleDrawFormat();
    this.direction = direction;
    this.drawPadding = drawPadding;
    if (direction > BOTTOM || direction < LEFT) {
        throw new TableException("Please set the direction less than 3 greater than 0");
    }
    rect = new Rect();
}

public int measureWidth2(Column column, TableConfig config) {
    Paint paint = config.getPaint();
    config.getContentStyle().fillPaint(paint);
    return DrawUtils.getMultiTextWidth(paint, getSplitString(column.getColumnName()));
}


@Override
public int measureWidth(Column column, TableConfig config) {
    int textWidth = measureWidth2(column, config);
    horizontalPadding = config.getColumnTitleHorizontalPadding();
    if (direction == LEFT || direction == RIGHT) {
        return getImageWidth() + textWidth + drawPadding;
    } else {
        return Math.max(super.measureWidth(column, config), textWidth);
    }
}

@Override
public int measureHeight(TableConfig config) {
    int imgHeight = super.measureHeight(config);
    **int textHeight = textDrawFormat.measureHeight(config) * 4;**
    verticalPadding = config.getColumnTitleVerticalPadding();

    if (direction == TOP || direction == BOTTOM) {
        return getImageHeight() + textHeight + drawPadding;
    } else {
        return Math.max(imgHeight, textHeight);
    }
}

@Override
public void draw(Canvas c, Column column, Rect rect, TableConfig config) {
    setDrawBackground(true);
    drawBackground(c, column, rect, config);
    setDrawBackground(false);
    textDrawFormat.setDrawBg(false);
    if (getBitmap(column) == null) {
        drawText(c, column, rect, config);
        return;
    }
    int width, imgLeft, imgRight, textWidth, height, imgTop, imgBottom, textHeight;
    switch (direction) {
        case LEFT:
            width = (int) (measureWidth(column, config) * config.getZoom());
            imgLeft = rect.left + (rect.right - rect.left - width) / 2;
            imgRight = (int) (imgLeft + getImageWidth() * config.getZoom());
            this.rect.set(imgLeft, rect.top, imgRight, rect.bottom);
            super.draw(c, column, this.rect, config);
            textWidth = (int) (measureWidth2(column, config) * config.getZoom());
            this.rect.set(imgRight + drawPadding, rect.top, imgRight + drawPadding + textWidth, rect.bottom);
            drawText(c, column, this.rect, config);
            break;
        case RIGHT:
            width = (int) (measureWidth(column, config) * config.getZoom());
            imgRight = rect.right - (rect.right - rect.left - width) / 2;
            imgLeft = (int) (imgRight - getImageWidth() * config.getZoom());
            this.rect.set(imgLeft, rect.top, imgRight, rect.bottom);
            super.draw(c, column, this.rect, config);
            textWidth = (int) (measureWidth2(column, config) * config.getZoom());
            this.rect.set(imgLeft - drawPadding - textWidth, rect.top, imgLeft - drawPadding, rect.bottom);
            drawText(c, column, this.rect, config);

            break;
        case TOP:
            height = (int) (measureHeight(config) * config.getZoom());
            imgTop = rect.top + (rect.top - rect.bottom - height) / 2;
            imgBottom = (int) (imgTop + getImageHeight() * config.getZoom());
            this.rect.set(rect.left, imgTop, rect.right, imgBottom);
            drawText(c, column, this.rect, config);
            textHeight = (int) (textDrawFormat.measureHeight(config) * config.getZoom());
            this.rect.set(rect.left, imgBottom + drawPadding, rect.right, imgBottom + drawPadding + textHeight);
            super.draw(c, column, this.rect, config);
            break;
        case BOTTOM:
            height = (int) (measureHeight(config) * config.getZoom());
            imgBottom = rect.bottom - (rect.bottom - rect.top - height) / 2;
            imgTop = (int) (imgBottom - getImageHeight() * config.getZoom());
            this.rect.set(rect.left, imgTop, rect.right, imgBottom);
            drawText(c, column, this.rect, config);
            textHeight = (int) (textDrawFormat.measureHeight(config) * config.getZoom());
            this.rect.set(rect.left, imgTop - drawPadding - textHeight, rect.right, imgTop - drawPadding);
            super.draw(c, column, this.rect, config);
            break;
    }
}

public void drawText(Canvas c,Column column, Rect rect,  TableConfig config) {
    Paint textPaint = config.getPaint();
    boolean isDrawBg = drawBackground(c, column, rect, config);
    config.getColumnTitleStyle().fillPaint(textPaint);
    ICellBackgroundFormat<Column> backgroundFormat = config.getColumnCellBackgroundFormat();

    textPaint.setTextSize(textPaint.getTextSize() * config.getZoom());
    if (isDrawBg && backgroundFormat.getTextColor(column) != TableConfig.INVALID_COLOR) {
        textPaint.setColor(backgroundFormat.getTextColor(column));
    }
    if (column.getTitleAlign() != null) { //如果列设置Align ，则使用列的Align
        textPaint.setTextAlign(column.getTitleAlign());
    }
    drawText2(c, column.getColumnName(), rect, textPaint);
}

protected void drawText2(Canvas c, String value, Rect rect, Paint paint) {
    DrawUtils.drawMultiText(c, paint, rect, getSplitString(value));
}

protected String[] getSplitString(String val) {
    String[] values = null;
    if (valueMap.get(val) != null) {
        values = valueMap.get(val).get();
    }
    if (values == null) {
        values = val.split("\n");
        valueMap.put(val, new SoftReference<>(values));
    }
    return values;
}
}

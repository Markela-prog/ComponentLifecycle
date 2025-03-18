## Component LifeCycle



### ngOnInit

The ngOnInit method runs after Angular has initialized all the components inputs with their initial values. A component's ngOnInit runs exactly once.

This step happens before the component's own template is initialized. This means that you can update the component's state based on its initial input values.

### ngOnChanges

The ngOnChanges method runs after any component inputs have changed.

This step happens before the component's own template is checked. This means that you can update the component's state based on its initial input values.

During initialization, the first ngOnChanges runs before ngOnInit.

### ngOnDestroy

The ngOnDestroy method runs once just before a component is destroyed. Angular destroys a component when it is no longer shown on the page, such as being hidden by @if or upon navigating to another page.

DestroyRef

As an alternative to the ngOnDestroy method, you can inject an instance of DestroyRef. You can register a callback to be invoked upon the component's destruction by calling the onDestroy method of DestroyRef.

```
@Component({
  /* ... */
})
export class UserProfile {
  constructor(private destroyRef: DestroyRef) {
    destroyRef.onDestroy(() => {
      console.log('UserProfile destruction');
    });
  }
}
```

You can pass the DestroyRef instance to functions or classes outside your component. Use this pattern if you have other code that should run some cleanup behavior when the component is destroyed.

You can also use DestroyRef to keep setup code close to cleanup code, rather than putting all cleanup code in the ngOnDestroy method.

### ngDoCheck

The ngDoCheck method runs before every time Angular checks a component's template for changes.

You can use this lifecycle hook to manually check for state changes outside of Angular's normal change detection, manually updating the component's state.

This method runs very frequently and can significantly impact your page's performance. Avoid defining this hook whenever possible, only using it when you have no alternative.

During initialization, the first ngDoCheck runs after ngOnInit.

### ngAfterContentInit

"Content" is some other (partial) View data structure projected into this component's View. (Basically ng-content)

The ngAfterContentInit method runs once after all the children nested inside the component (its content) have been initialized.

You can use this lifecycle hook to read the results of content queries. While you can access the initialized state of these queries, attempting to change any state in this method results in an ExpressionChangedAfterItHasBeenCheckedError

### ngAfterContentChecked

The ngAfterContentChecked method runs every time the children nested inside the component (its content) have been checked for changes.

This method runs very frequently and can significantly impact your page's performance. Avoid defining this hook whenever possible, only using it when you have no alternative.

While you can access the updated state of content queries here, attempting to change any state in this method results in an ExpressionChangedAfterItHasBeenCheckedError.

### ngAfterViewInit

Technically, the "View" is an internally managed data structure that holds references to the DOM elements rendered by a component. (Basically its template)

The ngAfterViewInit method runs once after all the children in the component's template (its view) have been initialized.

You can use this lifecycle hook to read the results of view queries. While you can access the initialized state of these queries, attempting to change any state in this method results in an ExpressionChangedAfterItHasBeenCheckedError

### ngAfterViewChecked

The ngAfterViewChecked method runs every time the children in the component's template (its view) have been checked for changes.

This method runs very frequently and can significantly impact your page's performance. Avoid defining this hook whenever possible, only using it when you have no alternative.

While you can access the updated state of view queries here, attempting to change any state in this method results in an ExpressionChangedAfterItHasBeenCheckedError.

### afterRender and afterNextRender

The afterRender and afterNextRender functions let you register a render callback to be invoked after Angular has finished rendering all components on the page into the DOM.

These functions are different from the other lifecycle hooks described in this guide. Rather than a class method, they are standalone functions that accept a callback. The execution of render callbacks are not tied to any specific component instance, but instead an application-wide hook.

afterRender and afterNextRender must be called in an injection context, typically a component's constructor.

You can use render callbacks to perform manual DOM operations. See Using DOM APIs for guidance on working with the DOM in Angular.

Render callbacks do not run during server-side rendering or during build-time pre-rendering.

<hr>

afterRender phases
When using afterRender or afterNextRender, you can optionally split the work into phases. The phase gives you control over the sequencing of DOM operations, letting you sequence write operations before read operations in order to minimize layout thrashing. In order to communicate across phases, a phase function may return a result value that can be accessed in the next phase.

```
import {Component, ElementRef, afterNextRender} from '@angular/core';
@Component({...})
export class UserProfile {
  private prevPadding = 0;
  private elementHeight = 0;
  constructor(elementRef: ElementRef) {
    const nativeElement = elementRef.nativeElement;
    afterNextRender({
      // Use the `Write` phase to write to a geometric property.
      write: () => {
        const padding = computePadding();
        const changed = padding !== this.prevPadding;
        if (changed) {
          nativeElement.style.padding = padding;
        }
        return changed; // Communicate whether anything changed to the read phase.
      },
      // Use the `Read` phase to read geometric properties after all writes have occurred.
      read: (didWrite) => {
        if (didWrite) {
          this.elementHeight = nativeElement.getBoundingClientRect().height;
        }
      }
    });
  }
}
```

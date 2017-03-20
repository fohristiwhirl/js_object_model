```python


# In Python...


class JS_Object:

    def __init__(self, prototype = None):       # When called with prototype, equivalent to Object.create()

        if prototype is not None:
            if type(prototype) is not JS_Object:
                raise TypeError

        object.__setattr__(self, "dict", {})
        object.__setattr__(self, "prototype", prototype)

    def __getattribute__(self, key):

        key = str(key)

        try:
            d = object.__getattribute__(self, "dict")
            return d[key]
        except KeyError:
            prototype = object.__getattribute__(self, "prototype")
            if prototype == None:
                return None                     # Strictly, should return some "undefined" object
            else:
                return prototype[key]

    def __setattr__(self, key, val):

        key = str(key)

        d = object.__getattribute__(self, "dict")
        d[key] = val

    def __getitem__(self, key):

        return object.__getattribute__(self, "__getattribute__")(key)

    def __setitem__(self, key, val):

        return object.__getattribute__(self, "__setattr__")(key, val)



class JS_Array (JS_Object):     # FIXME: handle the magical "length" property; add .push()

    @staticmethod
    def actualkey(key):         # Return an int if we should access the array, otherwise return a string

        if type(key) is int:
            return key
        elif type(key) is float:
            if int(key) == key:             # A float like 4.0 returns 4 (int)
                return int(key)
            else:                           # A float like 4.5 returns "4.5" (string)
                return str(key)
        elif type(key) is str:
            try:
                as_float = float(key)
                as_int = int(as_float)
                if as_float == as_int:      # A string like "4.0" returns 4 (int)       -- FIXME! This doesn't match JS
                    return as_int
                else:                       # A string like "4.5" returns "4.5" (string)
                    return key
            except:
                return key
        else:
            return str(key)

    def __init__(self, prototype = None):

        super().__init__(prototype)
        object.__setattr__(self, "array", {})       # Still a dictionary

    def __getattribute__(self, key):

        key = JS_Array.actualkey(key)

        if type(key) is int:

            try:
                a = object.__getattribute__(self, "array")
                return a[key]
            except KeyError:
                prototype = object.__getattribute__(self, "prototype")
                if prototype == None:
                    return None
                else:
                    return prototype[key]

        else:

            return super().__getattribute__(key)      # super() is the Python super, not the JS prototype

    def __setattr__(self, key, val):

        key = JS_Array.actualkey(key)

        if type(key) is int:

            d = object.__getattribute__(self, "array")
            d[key] = val

        else:

            super().__setattr__(key, val)             # super() is the Python super, not the JS prototype


```

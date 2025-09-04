import React, { useMemo, useState } from "react";
import { motion, AnimatePresence } from "framer-motion";
import {
  Search,
  ShoppingCart,
  MapPin,
  Bike,
  Clock,
  Star,
  ChevronLeft,
  Plus,
  Minus,
  CheckCircle,
  Heart,
} from "lucide-react";
import { Button } from "@/components/ui/button";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Input } from "@/components/ui/input";
import { Badge } from "@/components/ui/badge";
import { Sheet, SheetContent, SheetHeader, SheetTitle, SheetTrigger } from "@/components/ui/sheet";
import { Tabs, TabsContent, TabsList, TabsTrigger } from "@/components/ui/tabs";
import { Separator } from "@/components/ui/separator";

// --- Mock Data ---
const CATEGORIES = [
  { id: "pizza", label: "Pizza" },
  { id: "burger", label: "Burgers" },
  { id: "biryani", label: "Biryani" },
  { id: "dessert", label: "Desserts" },
  { id: "healthy", label: "Healthy" },
];

const RESTAURANTS = [
  {
    id: "r1",
    name: "Spicy Flame",
    cuisine: ["Indian", "Biryani"],
    rating: 4.6,
    eta: 30,
    fee: 6,
    banner:
      "https://images.unsplash.com/photo-1540189549336-e6e99c3679fe?q=80&w=1400&auto=format&fit=crop",
    items: [
      { id: "i1", name: "Chicken Biryani", price: 26, tag: "Popular" },
      { id: "i2", name: "Paneer Tikka", price: 22 },
      { id: "i3", name: "Butter Naan (2)", price: 8 },
    ],
  },
  {
    id: "r2",
    name: "Urban Slice",
    cuisine: ["Pizza", "Italian"],
    rating: 4.4,
    eta: 25,
    fee: 5,
    banner:
      "https://images.unsplash.com/photo-1548365328-9f547fb09530?q=80&w=1400&auto=format&fit=crop",
    items: [
      { id: "i4", name: "Margherita", price: 18, tag: "Chef's" },
      { id: "i5", name: "Pepperoni", price: 22 },
      { id: "i6", name: "Veggie Garden", price: 20 },
    ],
  },
  {
    id: "r3",
    name: "Burger Hub",
    cuisine: ["Burgers", "Fast Food"],
    rating: 4.2,
    eta: 20,
    fee: 4,
    banner:
      "https://images.unsplash.com/photo-1550547660-d9450f859349?q=80&w=1400&auto=format&fit=crop",
    items: [
      { id: "i7", name: "Classic Cheeseburger", price: 16, tag: "Best Seller" },
      { id: "i8", name: "Crispy Fries", price: 7 },
      { id: "i9", name: "Chocolate Shake", price: 10 },
    ],
  },
];

// --- Types ---
/** @typedef {{id: string, name: string, price: number, qty: number, restaurantId: string}} CartItem */

// --- Helpers ---
const currency = (n) => `QAR ${n.toFixed(2)}`;

function RestaurantCard({ r, onOpen }) {
  return (
    <motion.div layout>
      <Card className="overflow-hidden rounded-2xl shadow-sm hover:shadow-md transition-shadow cursor-pointer" onClick={() => onOpen(r)}>
        <div className="relative h-40 w-full">
          <img src={r.banner} alt={r.name} className="h-full w-full object-cover" />
          <Badge className="absolute top-2 left-2">{r.cuisine[0]}</Badge>
        </div>
        <CardContent className="p-4">
          <div className="flex items-center justify-between">
            <div>
              <div className="text-lg font-semibold">{r.name}</div>
              <div className="text-sm text-muted-foreground">{r.cuisine.join(" • ")}</div>
            </div>
            <div className="flex items-center gap-1 text-sm">
              <Star className="h-4 w-4" />
              <span className="font-medium">{r.rating}</span>
            </div>
          </div>
          <div className="mt-2 flex items-center gap-4 text-sm text-muted-foreground">
            <div className="flex items-center gap-1"><Clock className="h-4 w-4" />{r.eta}-{r.eta + 10} min</div>
            <div className="flex items-center gap-1"><Bike className="h-4 w-4" />Delivery {currency(r.fee)}</div>
          </div>
        </CardContent>
      </Card>
    </motion.div>
  );
}

function QtySelector({ qty, onDec, onInc }) {
  return (
    <div className="flex items-center gap-2">
      <Button size="icon" variant="outline" className="rounded-full" onClick={onDec}><Minus className="h-4 w-4" /></Button>
      <span className="min-w-6 text-center">{qty}</span>
      <Button size="icon" className="rounded-full" onClick={onInc}><Plus className="h-4 w-4" /></Button>
    </div>
  );
}

export default function App() {
  const [screen, setScreen] = useState("home"); // home | restaurant | checkout | track
  const [query, setQuery] = useState("");
  const [category, setCategory] = useState("");
  const [fav, setFav] = useState({});
  const [activeRestaurant, setActiveRestaurant] = useState(null);
  const [cart, setCart] = useState(/** @type {CartItem[]} */([]));
  const [address] = useState("Doha Corniche, Qatar");
  const [orderId, setOrderId] = useState(null);

  const filteredRestaurants = useMemo(() => {
    const q = query.toLowerCase();
    return RESTAURANTS.filter((r) => {
      const matchQ = !q || r.name.toLowerCase().includes(q) || r.cuisine.join(" ").toLowerCase().includes(q);
      const matchC = !category || r.cuisine.map((c) => c.toLowerCase()).includes(category);
      return matchQ && matchC;
    });
  }, [query, category]);

  const cartTotal = useMemo(() => cart.reduce((s, i) => s + i.price * i.qty, 0), [cart]);

  function addToCart(item, restaurantId) {
    setCart((prev) => {
      const idx = prev.findIndex((p) => p.id === item.id);
      if (idx !== -1) {
        const clone = [...prev];
        clone[idx] = { ...clone[idx], qty: clone[idx].qty + 1 };
        return clone;
      }
      return [...prev, { id: item.id, name: item.name, price: item.price, qty: 1, restaurantId }];
    });
  }
  function decFromCart(itemId) {
    setCart((prev) => {
      const idx = prev.findIndex((p) => p.id === itemId);
      if (idx === -1) return prev;
      const clone = [...prev];
      const q = clone[idx].qty - 1;
      if (q <= 0) return clone.filter((x) => x.id !== itemId);
      clone[idx] = { ...clone[idx], qty: q };
      return clone;
    });
  }

  function clearCart() { setCart([]); }

  function openRestaurant(r) {
    setActiveRestaurant(r);
    setScreen("restaurant");
  }

  function placeOrder() {
    // In real app, call backend then navigate to tracking
    const oid = `ORD-${Math.random().toString(36).slice(2, 7).toUpperCase()}`;
    setOrderId(oid);
    setScreen("track");
  }

  return (
    <div className="min-h-screen bg-gradient-to-b from-white to-slate-50 text-slate-900">
      {/* Header */}
      <div className="sticky top-0 z-40 bg-white/80 backdrop-blur border-b">
        <div className="mx-auto max-w-5xl px-4 py-3 flex items-center gap-3">
          {screen !== "home" && (
            <Button variant="ghost" size="icon" onClick={() => setScreen("home")} className="rounded-full"><ChevronLeft className="h-5 w-5" /></Button>
          )}
          <MapPin className="h-5 w-5" />
          <div className="text-sm">
            <div className="font-semibold leading-4">Deliver to</div>
            <div className="text-muted-foreground leading-4">{address}</div>
          </div>
          <div className="ml-auto flex items-center gap-2">
            <Sheet>
              <SheetTrigger asChild>
                <Button variant="outline" className="rounded-full"><ShoppingCart className="h-4 w-4 mr-2" />{cart.length ? `${cart.length} items` : "Cart"}</Button>
              </SheetTrigger>
              <SheetContent className="w-full sm:max-w-md">
                <SheetHeader>
                  <SheetTitle>Your Cart</SheetTitle>
                </SheetHeader>
                <div className="mt-4 space-y-4">
                  {cart.length === 0 && <div className="text-sm text-muted-foreground">Your cart is empty.</div>}
                  {cart.map((c) => (
                    <div key={c.id} className="flex items-center justify-between">
                      <div>
                        <div className="font-medium">{c.name}</div>
                        <div className="text-sm text-muted-foreground">{currency(c.price)}</div>
                      </div>
                      <QtySelector qty={c.qty} onDec={() => decFromCart(c.id)} onInc={() => addToCart(c, c.restaurantId)} />
                    </div>
                  ))}
                </div>
                <Separator className="my-4" />
                <div className="flex items-center justify-between mb-4">
                  <div className="font-semibold">Subtotal</div>
                  <div>{currency(cartTotal)}</div>
                </div>
                <div className="flex items-center justify-between text-sm text-muted-foreground mb-6">
                  <div>Delivery</div>
                  <div>{currency(activeRestaurant?.fee ?? 0)}</div>
                </div>
                <Button className="w-full rounded-full" disabled={!cart.length} onClick={() => setScreen("checkout")}>Checkout</Button>
                <Button variant="ghost" className="w-full mt-2" onClick={clearCart}>Clear Cart</Button>
              </SheetContent>
            </Sheet>
          </div>
        </div>
      </div>

      {/* Search & Categories (Home Only) */}
      {screen === "home" && (
        <div className="mx-auto max-w-5xl px-4 py-6">
          <div className="flex gap-2 items-center">
            <div className="relative w-full">
              <Search className="absolute left-3 top-1/2 -translate-y-1/2 h-4 w-4" />
              <Input placeholder="Search restaurants, cuisines..." className="pl-10 rounded-full" value={query} onChange={(e) => setQuery(e.target.value)} />
            </div>
          </div>

          <Tabs className="mt-4" value={category || "all"} onValueChange={(v) => setCategory(v === "all" ? "" : v)}>
            <TabsList className="flex flex-wrap justify-start gap-2 bg-transparent p-0">
              <TabsTrigger value="all" className="rounded-full border px-4">All</TabsTrigger>
              {CATEGORIES.map((c) => (
                <TabsTrigger key={c.id} value={c.id} className="rounded-full border px-4">{c.label}</TabsTrigger>
              ))}
            </TabsList>
            <TabsContent value={category || "all"}>
              <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4 mt-4">
                <AnimatePresence>
                  {filteredRestaurants.map((r) => (
                    <motion.div
                      key={r.id}
                      layout
                      initial={{ opacity: 0, y: 10 }}
                      animate={{ opacity: 1, y: 0 }}
                      exit={{ opacity: 0, y: -10 }}
                      transition={{ duration: 0.2 }}
                    >
                      <RestaurantCard r={r} onOpen={openRestaurant} />
                    </motion.div>
                  ))}
                </AnimatePresence>
              </div>
            </TabsContent>
          </Tabs>
        </div>
      )}

      {/* Restaurant Page */}
      {screen === "restaurant" && activeRestaurant && (
        <div className="mx-auto max-w-3xl px-4 pb-20">
          <motion.div initial={{ opacity: 0 }} animate={{ opacity: 1 }}>
            <div className="relative h-56 w-full mt-4 overflow-hidden rounded-2xl">
              <img src={activeRestaurant.banner} alt={activeRestaurant.name} className="h-full w-full object-cover" />
              <Button
                size="icon"
                variant="secondary"
                className="absolute top-3 right-3 rounded-full"
                onClick={() => setFav((f) => ({ ...f, [activeRestaurant.id]: !f[activeRestaurant.id] }))}
              >
                <Heart className={`h-4 w-4 ${fav[activeRestaurant.id] ? "fill-current" : ""}`} />
              </Button>
            </div>

            <div className="mt-4 flex items-start justify-between">
              <div>
                <h1 className="text-2xl font-bold">{activeRestaurant.name}</h1>
                <div className="text-sm text-muted-foreground mt-1">{activeRestaurant.cuisine.join(" • ")}</div>
                <div className="flex items-center gap-3 text-sm mt-2">
                  <div className="flex items-center gap-1"><Star className="h-4 w-4" />{activeRestaurant.rating}</div>
                  <div className="flex items-center gap-1"><Clock className="h-4 w-4" />{activeRestaurant.eta}-{activeRestaurant.eta + 10} min</div>
                  <div className="flex items-center gap-1"><Bike className="h-4 w-4" />{currency(activeRestaurant.fee)} fee</div>
                </div>
              </div>
              <div>
                <Button className="rounded-full" onClick={() => setScreen("home")}>Browse more</Button>
              </div>
            </div>

            <div className="mt-6 space-y-3">
              {activeRestaurant.items.map((it) => (
                <Card key={it.id} className="rounded-2xl">
                  <CardHeader className="py-3">
                    <div className="flex items-center justify-between">
                      <div>
                        <CardTitle className="text-base">{it.name}</CardTitle>
                        <div className="text-sm text-muted-foreground">{currency(it.price)}</div>
                        {it.tag && <Badge className="mt-1">{it.tag}</Badge>}
                      </div>
                      <div className="flex items-center gap-2">
                        <Button className="rounded-full" onClick={() => addToCart(it, activeRestaurant.id)}>
                          <Plus className="h-4 w-4 mr-1" /> Add
                        </Button>
                      </div>
                    </div>
                  </CardHeader>
                </Card>
              ))}
            </div>
          </motion.div>
        </div>
      )}

      {/* Checkout */}
      {screen === "checkout" && (
        <div className="mx-auto max-w-3xl px-4 pb-24">
          <h2 className="mt-6 text-2xl font-bold">Checkout</h2>
          <div className="mt-4 grid grid-cols-1 md:grid-cols-3 gap-4">
            <Card className="md:col-span-2 rounded-2xl">
              <CardHeader>
                <CardTitle>Delivery Details</CardTitle>
              </CardHeader>
              <CardContent className="space-y-3">
                <Input placeholder="Full Name" className="rounded-xl" defaultValue="Mr Ishtiaq Ali" />
                <Input placeholder="Mobile Number" className="rounded-xl" defaultValue="+974 55 555 555" />
                <Input placeholder="Address" className="rounded-xl" defaultValue={address} />
                <Input placeholder="Note to rider (optional)" className="rounded-xl" />
              </CardContent>
            </Card>
            <Card className="rounded-2xl">
              <CardHeader>
                <CardTitle>Order Summary</CardTitle>
              </CardHeader>
              <CardContent>
                <div className="space-y-3">
                  {cart.map((c) => (
                    <div key={c.id} className="flex items-center justify-between text-sm">
                      <div>
                        <div className="font-medium">{c.name}</div>
                        <div className="text-muted-foreground">x{c.qty}</div>
                      </div>
                      <div>{currency(c.price * c.qty)}</div>
                    </div>
                  ))}
                </div>
                <Separator className="my-4" />
                <div className="flex items-center justify-between text-sm mb-1"><span>Subtotal</span><span>{currency(cartTotal)}</span></div>
                <div className="flex items-center justify-between text-sm mb-1"><span>Delivery</span><span>{currency(activeRestaurant?.fee ?? 0)}</span></div>
                <div className="flex items-center justify-between font-semibold text-base mt-2">
                  <span>Total</span>
                  <span>{currency(cartTotal + (activeRestaurant?.fee ?? 0))}</span>
                </div>
                <Button className="w-full mt-4 rounded-full" disabled={!cart.length} onClick={placeOrder}><CheckCircle className="h-4 w-4 mr-1" /> Place Order</Button>
              </CardContent>
            </Card>
          </div>
        </div>
      )}

      {/* Order Tracking */}
      {screen === "track" && (
        <div className="mx-auto max-w-3xl px-4 pb-24">
          <h2 className="mt-6 text-2xl font-bold">Order Tracking</h2>
          <Card className="mt-4 rounded-2xl">
            <CardContent className="p-6">
              <div className="flex items-center justify-between">
                <div>
                  <div className="text-sm text-muted-foreground">Order ID</div>
                  <div className="font-semibold">{orderId}</div>
                </div>
                <Badge variant="secondary">On the way</Badge>
              </div>
              <div className="mt-6 grid grid-cols-1 md:grid-cols-3 gap-4">
                {[
                  { step: 1, title: "Order Confirmed", desc: "Restaurant accepted your order." },
                  { step: 2, title: "Preparing", desc: "Your food is being prepared." },
                  { step: 3, title: "Out for Delivery", desc: "Rider is heading to you." },
                ].map((s, idx) => (
                  <div key={s.step} className={`p-4 border rounded-xl ${idx <= 2 ? "bg-white" : "opacity-50"}`}>
                    <div className="flex items-center gap-2">
                      <CheckCircle className="h-4 w-4" />
                      <div className="font-medium">{s.title}</div>
                    </div>
                    <div className="text-sm text-muted-foreground mt-1">{s.desc}</div>
                  </div>
                ))}
              </div>
              <div className="mt-6 flex items-center gap-2 text-sm text-muted-foreground">
                <Bike className="h-4 w-4" /> ETA ~ {activeRestaurant ? activeRestaurant.eta : 30}-{activeRestaurant ? activeRestaurant.eta + 10 : 40} min
              </div>
              <div className="mt-6 flex items-center gap-2">
                <Button variant="outline" className="rounded-full" onClick={() => setScreen("home")}>Back to Home</Button>
              </div>
            </CardContent>
          </Card>
        </div>
      )}

      {/* Bottom Nav */}
      <div className="fixed bottom-4 left-1/2 -translate-x-1/2">
        <div className="bg-white border shadow-lg rounded-full px-3 py-2 flex items-center gap-2">
          <Button variant={screen === "home" ? "default" : "ghost"} className="rounded-full" onClick={() => setScreen("home")}>Home</Button>
          <Button variant={screen === "checkout" ? "default" : "ghost"} className="rounded-full" onClick={() => setScreen("checkout")} disabled={!cart.length}>Checkout</Button>
          <Button variant={screen === "track" ? "default" : "ghost"} className="rounded-full" onClick={() => setScreen("track")} disabled={!orderId}>Track</Button>
        </div>
      </div>

      {/* Footer */}
      <div className="py-16" />
    </div>
  );
}

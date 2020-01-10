#### BITS



```
template <class T>
void runMultiBitTest8() {
  auto load = detail::BitsTraits<T>::load;
  T buf[] = {0x12, 0x34, 0x56, 0x78};

  // 0100 1000 0010 1100 0110 1010 0001 1110
  EXPECT_EQ(0x02, load(Bits<T>::get(buf, 0, 4)));
  EXPECT_EQ(0x1a, load(Bits<T>::get(buf, 9, 5))); // 0001 1010
  EXPECT_EQ(0xb1, load(Bits<T>::get(buf, 13, 8)));

  Bits<T>::set(buf, 0, 4, 0x0b);
  EXPECT_EQ(0x1b, load(buf[0]));
  EXPECT_EQ(0x34, load(buf[1]));
  EXPECT_EQ(0x56, load(buf[2]));
  EXPECT_EQ(0x78, load(buf[3]));

  Bits<T>::set(buf, 9, 5, 0x0e);
  EXPECT_EQ(0x1b, load(buf[0]));
  EXPECT_EQ(0x1c, load(buf[1]));
  EXPECT_EQ(0x56, load(buf[2]));
  EXPECT_EQ(0x78, load(buf[3]));

  Bits<T>::set(buf, 13, 8, 0xaa);
  EXPECT_EQ(0x1b, load(buf[0]));
  EXPECT_EQ(0x5c, load(buf[1]));
  EXPECT_EQ(0x55, load(buf[2]));
  EXPECT_EQ(0x78, load(buf[3]));
}
```


<script lang="ts">
import { defineComponent, type PropType } from 'vue';
import CardInfoComponent, { CardLayoutType } from '@/components/common/CardInfoComponent.vue';
import SliderComponent from '@/components/common/SliderComponent.vue';
import type Card from '@/interfaces/card';
interface Filter {
  id: number;
  name: string;
  img: string;
  imgWhite: string;
}

export default defineComponent({
  data() {
    return {
      filters: [
        {
          id: 1,
          name: 'all',
          img: 'src/assets/images/filter/board_cards.png',
          imgWhite: 'src/assets/images/filter/board_cards_white.png',
        },
        {
          id: 2,
          name: 'melee',
          img: 'src/assets/images/filter/melee_tan.png',
          imgWhite: 'src/assets/images/filter/melee_white.png',
        },
        {
          id: 3,
          name: 'range',
          img: 'src/assets/images/filter/ranged_tan.png',
          imgWhite: 'src/assets/images/filter/ranged_white.png',
        },
        {
          id: 4,
          name: 'siege',
          img: 'src/assets/images/filter/siege_tan.png',
          imgWhite: 'src/assets/images/filter/siege_white.png',
        },
        {
          id: 5,
          name: 'hero',
          img: 'src/assets/images/filter/hero_tan.png',
          imgWhite: 'src/assets/images/filter/hero_white.png',
        },
        {
          id: 6,
          name: 'weather',
          img: 'src/assets/images/filter/sky_tan.png',
          imgWhite: 'src/assets/images/filter/sky_white.png',
        },
        {
          id: 7,
          name: 'special',
          img: 'src/assets/images/filter/dummy_tan.png',
          imgWhite: 'src/assets/images/filter/dummy_white.png',
        },
      ] as Filter[],
      currFilter: 'all',

      cardLayoutType: CardLayoutType,
      selectedSliderCard: [] as Card[],
      isShowSlider: false,
    };
  },
  methods: {
    changeFilter(filter: Filter): void {
      this.currFilter = filter.name;
      this.$emit('filterChanged', this.currFilter);
    },
    takeCard(item: Card) {
      this.$emit('selectedCard', item);
    },
    showSlider(card: Card, event: Event) {
      event.preventDefault();
      this.selectedSliderCard = [];
      this.selectedSliderCard.push(card);
      this.isShowSlider = true;
    },
  },
  computed: {
    filterFractions() {
      return this.gwentCards.filter(
        (item) => item.fractionId === null || (item.fractionId === this.currentFraction && item.type !== 'leader')
      );
    },
    filteredCards() {
      if (this.currFilter === 'all') {
        return this.filterFractions.filter(
          (item) => item.fractionId === null || item.fractionId === this.currentFraction
        );
      } else if (this.currFilter === 'hero' || this.currFilter === 'special') {
        return this.filterFractions.filter((card) => card.type === this.currFilter);
      } else if (this.currFilter === 'weather') {
        return this.filterFractions.filter((card) => card.fieldType.some((type) => type === this.currFilter));
      }
      return this.filterFractions.filter(
        (card) =>
          card.fieldType.some((type) => type === this.currFilter) && (card.type === 'usual' || card.type === 'hero')
      );
    },
  },
  components: {
    CardInfoComponent,
    SliderComponent,
  },
  props: {
    currentFraction: {
      type: Number,
    },
    gwentCards: {
      type: Array as PropType<Card[]>,
      required: true,
    },
  },
});
</script>

<template>
  <div class="collection">
    <div class="collection__filter">
      <img
        class="filter__img"
        v-for:="filter in filters"
        :src="currFilter === filter.name ? filter.imgWhite : filter.img"
        @click="changeFilter(filter)"
        alt="сортировка"
        draggable="false"
      />
    </div>
    <div class="cards">
      <div class="card" v-for="item in filteredCards" :key="item.id">
        <CardInfoComponent
          :card="item"
          :layout-type="cardLayoutType.AVERAGE"
          @click="takeCard(item)"
          @contextmenu="showSlider(item, $event)"
        />
      </div>
    </div>
    <div v-if="isShowSlider" class="slider__wrap" @click="isShowSlider = false">
      <SliderComponent :cards="selectedSliderCard" />
    </div>
  </div>
</template>

<style lang="scss" scoped>
.slider__wrap {
  position: absolute;
  z-index: 10;
  height: 100vh;
  width: 100vw;
  padding-top: 7vw;
  top: -2.9vw;
  left: -7.5vw;
  background-color: rgba($color: #000000, $alpha: 0.7);
}
.collection {
  width: 40%;
  height: 100%;
  &__filter {
    height: 8%;
    margin-top: 2%;
    display: flex;
    justify-content: space-between;
    border-bottom: 1px rgba($color: #4d392c, $alpha: 0.5) solid;
  }
}

.filter__img {
  width: 10%;
  transition: 0.1s;
  object-fit: contain;

  &:hover {
    transform: scale(1.1);
  }
}

.cards {
  margin-top: 1%;
  height: 39.5vw;
  display: flex;
  flex-wrap: wrap;
  gap: 1vw 1.2vw;
  padding: 3%;
  padding-top: 2%;
  align-content: flex-start;
  overflow: auto;

  &::-webkit-scrollbar {
    width: 0.4vw;
  }
  &::-webkit-scrollbar-thumb {
    background-color: rgba($color: #4d392c, $alpha: 0.2);
    border: 1px rgba($color: #4d392c, $alpha: 0.6) solid;
    border-radius: 20px;
  }
}

.card {
  box-sizing: border-box;
  position: relative;
  width: 9.45vw;
  height: 17.6vw;
  border-radius: 1vw;

  &:hover {
    animation: pulse 2s infinite;
  }
}
</style>
